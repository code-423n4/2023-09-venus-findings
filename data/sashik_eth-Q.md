## LOW-1 PrimeLiquidityProvider#releaseFunds function could revert 

`releaseFunds` function does not check if the current accrued amount is zero and calls `safeTransfer` in any case:

```solidity 
File: PrimeLiquidityProvider.sol
192:     function releaseFunds(address token_) external {
193:         if (msg.sender != prime) revert InvalidCaller();
...
199:         uint256 accruedAmount = tokenAmountAccrued[token_];
200:         tokenAmountAccrued[token_] = 0;
...
204:         IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);
205:     }
```

So, in case the reward token does not allow 0 zero transfers (for example, [BNB token on mainnet](https://etherscan.io/token/0xB8c77482e45F1F44dE1745F52C74426C631bDD52#code)) and the current accrued amount is zero, this function would revert causing DOS on external caller - `Prime#_claimInterest`.

Recommendation: Consider adding a check that the transferred amount is > 0 and transfer tokens only in this case.

## LOW-2 Using `block.number` for reward calculation may pose security risks on Layer 2 (L2) chains.

In the `PrimeLiquidityProvider` contract, the `accrueTokens` function utilizes the `block.number` variable to calculate accrued token amounts. This is achieved by tracking the number of blocks elapsed since the last accrual:
```solidty
File: PrimeLiquidityProvider.sol
249:     function accrueTokens(address token_) public {
...
254:         uint256 blockNumber = getBlockNumber();
255:         uint256 deltaBlocks = blockNumber - lastAccruedBlock[token_];
...
```

Also, the `Prime` contract includes a `BLOCKS_PER_YEAR` immutable value that represents the expected number of blocks that would be produced on the current chain. Later this immutable variable was used in `_incomeDistributionYearly` function:
```solidity
File: Prime.sol
970:     function _incomeDistributionYearly(address vToken) internal view returns (uint256 amount) {
971:         uint256 totalIncomePerBlockFromMarket = _incomePerBlock(vToken);
972:         uint256 incomePerBlockForDistributionFromMarket = (totalIncomePerBlockFromMarket * _distributionPercentage()) /
973:             IProtocolShareReserve(protocolShareReserve).MAX_PERCENT();
974:         amount = BLOCKS_PER_YEAR * incomePerBlockForDistributionFromMarket;
975: 
976:         uint256 totalIncomePerBlockFromPLP = IPrimeLiquidityProvider(primeLiquidityProvider)
977:             .getEffectiveDistributionSpeed(_getUnderlying(vToken));
978:         amount += BLOCKS_PER_YEAR * totalIncomePerBlockFromPLP;
979:     }
```

While it is a common practice to use `block.number` (or expected rate of blocks per period) for various accounting purposes, it's important to note that this approach may become problematic on Layer 2 chains. The behavior of `block.number` on Layer 2 chains can differ significantly from the Ethereum mainnet, and changes in this behavior may occur in the future. For instance, zkSync recently experienced changes in `block.number` production rates, as documented here: https://github.com/zkSync-Community-Hub/zkync-developers/discussions/87

Recommendation: Consider using `block.timestamp` as a measure of time instead of `block.number` for reward calculations. Unlike `block.number`, `block.timestamp` is less likely to exhibit significant variations, even in the event of changes on the underlying blockchain. 

## LOW-3 Potential for unauthorized early interest claims using `Prime#claimInterest`

The `Prime#claimInterest` function allows for the claiming of interest on behalf of an arbitrary user address:
```solidity
File: Prime.sol
443:     function claimInterest(address vToken, address user) external whenNotPaused returns (uint256) { 
444:         return _claimInterest(vToken, user);
445:     }
```

While this function does not permit attackers to steal or lock a user's interest, it does allow them to influence the timing of when interest is claimed. This capability could potentially create issues in scenarios such as tax accounting, where the date of funds arrival affects the amount of taxes to be paid.

Recommendation: Consider to restrict the use of the Prime#claimInterest function to only the authorized owner or designated users.

## LOW-4 Attacker could potentially avoid rewardIndex increase in `Prime#accrueInterest` function

The Prime#accrueInterest function is responsible for accounting accrued interest on each vToken. It increases the rewardIndex for each market by calculating a delta value based on distributionIncome and the sumOfMembersScore of the market. However, there is a potential issue where, if distributionIncome is sufficiently small while sumOfMembersScore is large, the calculation at line 585 could result in a delta value of 0. This means that the rewardIndex would not increase, even though unreleased amounts are properly accounted for. A malicious actor could exploit this by frequently calling the accrueInterest function to prevent rewardIndex from increasing, potentially interfering with the distribution of rewards to Prime holders:
```solidity
File: Prime.sol
554:     function accrueInterest(address vToken) public {
...
568:         uint256 distributionIncome = totalIncomeUnreleased - unreleasedPSRIncome[underlying];
569: 
570:         _primeLiquidityProvider.accrueTokens(underlying);
571:         uint256 totalAccruedInPLP = _primeLiquidityProvider.tokenAmountAccrued(underlying);
572:         uint256 unreleasedPLPAccruedInterest = totalAccruedInPLP - unreleasedPLPIncome[underlying];
573: 
574:         distributionIncome += unreleasedPLPAccruedInterest;
...
580:         unreleasedPSRIncome[underlying] = totalIncomeUnreleased;
581:         unreleasedPLPIncome[underlying] = totalAccruedInPLP;
582:
583:         uint256 delta;
584:         if (markets[vToken].sumOfMembersScore > 0) {
585:             delta = ((distributionIncome * EXP_SCALE) / markets[vToken].sumOfMembersScore); // @audit-issue medium? could be 0 ever?
586:         }
587: 
588:         markets[vToken].rewardIndex = markets[vToken].rewardIndex + delta;
589:     }
```

Recommendation: Consider updating `unreleasedPSRIncome` and `unreleasedPLPIncome` only in cases if delta > 0.

## LOW-5 Attacker could potentially avoid accrued amount increase in Prime#_executeBoost function

The `Prime#_executeBoost` function is responsible for executing a boost for a user within a specific vToken market. It accrues interest for the user and updates their accrued interest based on `rewardIndex` delta and the user's score. However, there is a potential issue where, if the delta of `rewardIndex` change multiplied by the user's score is less than `EXP_SCALE`, the attacker could frequently call the `accrueInterestAndUpdateScore` function and avoid the user from accruing rewards.

```solidity
File: Prime.sol
918:     function _interestAccrued(address vToken, address user) internal view returns (uint256) {
919:         uint256 index = markets[vToken].rewardIndex - interests[vToken][user].rewardIndex;
920:         uint256 score = interests[vToken][user].score;
921: 
922:         return (index * score) / EXP_SCALE; 
923:     }
924: 
...
779:     function _executeBoost(address user, address vToken) internal {
780:         if (!markets[vToken].exists || !tokens[user].exists) {
781:             return;
782:         }
783: 
784:         accrueInterest(vToken);
785:         interests[vToken][user].accrued += _interestAccrued(vToken, user);
786:         interests[vToken][user].rewardIndex = markets[vToken].rewardIndex;
787:     }
``` 

In case if delta of rewardIndex change multiplied to user score is less than `EXP_SCALE` Attacker could frequently call `accrueInterestAndUpdateScore` function and avoid user from accruing rewards.

Recommendation: consider updating the `_executeBoost` function to update the `rewardIndex` value only if the accrued amount for the user is greater than zero.