## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | Misleading comments in `PrimeLiquidityProvider.sol` | 2 |
| [L&#x2011;02] | `BLOCKS_PER_YEAR` should not be immutable | 1 |
| [L&#x2011;03] | `α` (alpha) will be default to 0.5 and won't be changed | 1 |
| [L&#x2011;04] | Insufficient price validation from oracle in `Prime.sol` | 1 |
| [L&#x2011;05] | msg.sender as recipient address should be set in `releaseFunds()` while transfering tokens | 1 |
| [L&#x2011;06] | Misleading comment in `Prime.sol` | 1 |
| [L&#x2011;07] | External function should be called at last and should not violate CEI pattern | 1 |

### [L&#x2011;01]  Misleading comments in `PrimeLiquidityProvider.sol`
In `PrimeLiquidityProvider.sol`, There are below misleading comments. There is governance control or governance address access to functions in this contract. However the comments mention `onlyGovernance` which is misleading and it should be `onlyOwner` since these functions can only be accessed by owner of contract.

There are 2 instances of this issues:

### Recommended Mitigation steps

```diff

-     * @custom:access Only Governance
+     * @custom:access Only Owner
     */
118    function initializeTokens(address[] calldata tokens_) external onlyOwner {

      // some code

-     * @custom:access Only Governance
+     * @custom:access Only Owner
     */
216    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
```

### [L&#x2011;02]  `BLOCKS_PER_YEAR` should not be immutable address
In `Prime.sol`, `BLOCKS_PER_YEAR` is considered to be immutable which means it can not be changed later. venus prime intends to be deployed on `BNB Chain, Ethereum mainnet, Arbitrum, Polygon zkEVM, opBNB`. BLOCKS_PER_YEAR is calculated considering the current block time of the particular chain either in this case L1 or L2 chains. As seen in past, block durations has been changed via forks and it is evident that it will further change in future.

Since venus prime also intends to be deployed on `Ethereum mainnet`. Ethereum will have line of upgrades till 2025 and it is possible that the block time from 12 seconds will be further changed in next few years. Therefore the calculations of `BLOCKS_PER_YEAR` will be largely affected. 

While calulating `_incomeDistributionYearly()` will increase multifold as due to decrease in block time, the number of blocks will be increase and to overcome it the `BLOCKS_PER_YEAR` will need to be updated so that the accounting errors/loss can be prevented. Therefore `BLOCKS_PER_YEAR` should not be made immutable.

### Recommended Mitigation Steps
`BLOCKS_PER_YEAR` should be changeable and should be separate function which can be accessed by `onlyAdmin`. 


### [L&#x2011;03]  `α`(alpha) will be default to 0.5 and won't be changed
Per the documentation,
> A default weight of 0.5 weight has been evaluated as a good ratio and is not likely to be changed. 

However, while passing the value of `_alphaNumerator` and `_alphaDenominator`, the ratio is not validated to be 0.5 as this will be fixed and wont be changed.

There is 1 instance of this issue which can be checked [here](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L809C68-L813)

```Solidity
File: contracts/Tokens/Prime/Prime.sol

    function _checkAlphaArguments(uint128 _alphaNumerator, uint128 _alphaDenominator) internal {
        if (_alphaDenominator == 0 || _alphaNumerator > _alphaDenominator) {
            revert InvalidAlphaArguments();
        }
    }
```

### Recommended Mitigation steps
Add validation so that the `α`(alpha) should not be more than 0.5

### [L&#x2011;04]  Insufficient price validation from oracle in `Prime.sol`
In `Prime.sol`, The price from oracle is fetched from `ResilientOracle`. The issue here is the price returned from the oracle is not validated and it is possible that oracle can return 0 price which will further lead the calculations to 0 because oracle price is getting multiplied which can be seen in code.

There is 2 instances of this issue which can be checked [here](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L880-L884)

### Recommended Mitigation Steps

```diff
File: contracts/Tokens/Prime/Prime.sol

    function _capitalForScore(
        uint256 xvs,
        uint256 borrow,
        uint256 supply,
        address market
    ) internal view returns (uint256, uint256, uint256) {
        address xvsToken = IXVSVault(xvsVault).xvsAddress();

        uint256 xvsPrice = oracle.getPrice(xvsToken);
+       require(xvsPrice != 0, "invalid xvsPrice");
        uint256 borrowCapUSD = (xvsPrice * ((xvs * markets[market].borrowMultiplier) / EXP_SCALE)) / EXP_SCALE;
        uint256 supplyCapUSD = (xvsPrice * ((xvs * markets[market].supplyMultiplier) / EXP_SCALE)) / EXP_SCALE;

        uint256 tokenPrice = oracle.getUnderlyingPrice(market);
+       require(tokenPrice != 0, "invalid tokenPrice");
        uint256 supplyUSD = (tokenPrice * supply) / EXP_SCALE;
        uint256 borrowUSD = (tokenPrice * borrow) / EXP_SCALE;
```

| [L&#x2011;05] | msg.sender as recipient address should be set in `releaseFunds()` while transfering tokens | 1 |

In `PrimeLiquidityProvider.sol`, `releaseFunds()` can only be accessed by `Prime` contract. While transfering the funds to prime contract the recipient address be set to `msg.sender` as it prime contract is the only msg.sender which can call this function.

There is 1 instance of this issue which can be checked [here](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L204)

```Solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

    function releaseFunds(address token_) external {
        if (msg.sender != prime) revert InvalidCaller();
        if (paused()) {
            revert FundsTransferIsPaused();
        }

        accrueTokens(token_);
        uint256 accruedAmount = tokenAmountAccrued[token_];
        tokenAmountAccrued[token_] = 0;

        emit TokenTransferredToPrime(token_, accruedAmount);

        IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);
    }
```

### Recommended Mitigation steps

```diff
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

    function releaseFunds(address token_) external {
        if (msg.sender != prime) revert InvalidCaller();
        if (paused()) {
            revert FundsTransferIsPaused();
        }

        accrueTokens(token_);
        uint256 accruedAmount = tokenAmountAccrued[token_];
        tokenAmountAccrued[token_] = 0;

        emit TokenTransferredToPrime(token_, accruedAmount);

-        IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);
+        IERC20Upgradeable(token_).safeTransfer(msg.sender, accruedAmount);
    }
```

### [L&#x2011;06]  Misleading comment in `Prime.sol`
Following comment should be corrected which may mislead code readers.

There is 1 instances of this issue which can be checked [here](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L385)

```diff
-     * @notice accrues interes and updates score for an user for a specific market
+     * @notice accrues interest and updates score for an user for a specific market
```

### [L&#x2011;07]  External function should be called at last and should not violate CEI pattern
Checks, Effects and Interactions(CEI) pattern should be followed and external functions should be called at last of functions and state should be updated first.

There is 1 instance of this issue which can be checked [here](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L688)

### Recommended Mitigation steps

```diff
File: 

    function _claimInterest(address vToken, address user) internal returns (uint256) {
        uint256 amount = getInterestAccrued(vToken, user);
        amount += interests[vToken][user].accrued;

        interests[vToken][user].rewardIndex = markets[vToken].rewardIndex;
        interests[vToken][user].accrued = 0;

        address underlying = _getUnderlying(vToken);
        IERC20Upgradeable asset = IERC20Upgradeable(underlying);

        if (amount > asset.balanceOf(address(this))) {
            address[] memory assets = new address[](1);
            assets[0] = address(asset);
            IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
            if (amount > asset.balanceOf(address(this))) {
+                unreleasedPLPIncome[underlying] = 0;
                 IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
-                unreleasedPLPIncome[underlying] = 0;
            }
        }

        asset.safeTransfer(user, amount);

        emit InterestClaimed(user, vToken, amount);

        return amount;
    }
```