### [Low-01]: Adversary can prevent the issuing of prime token, by frontrunning with `claim` call
The `issue` call takes users[] as input, to which the revocable/irrevocable token to mint. If the users arr include the address which is already eligible for revocable tokens. Then, any `issue` call for revocable token can be prevented with frontrunning `claim` call by the adversary. 

```solidity
   function claim() external {
        if (stakedAt[msg.sender] == 0) revert IneligibleToClaim();
        if (block.timestamp - stakedAt[msg.sender] < STAKING_PERIOD) revert WaitMoreTime();

        stakedAt[msg.sender] = 0;

        _mint(false, msg.sender);
        _initializeMarkets(msg.sender);
    }
```
The `claim` internally call `_mint` function, which changes `tokens[user].exists = true`, it also make sure if the token minted for same user again, it revert in first line. 

```solidity
    function _mint(bool isIrrevocable, address user) internal {
        if (tokens[user].exists) revert IneligibleToClaim(); 
        
        tokens[user].exists = true; 
        // ...SNIP...
    }
```

When the `issue` iterate over array of users for the adversary address, the txn will revert with `IneligibleToClaim` err, bc the token for adversary already exists. 

**Recommendation**
*File: Prime.sol*
```solidity
    function issue(bool isIrrevocable, address[] calldata users) external {
        _checkAccessAllowed("issue(bool,address[])");

        if (isIrrevocable) {
        // ...SNIP...
        } else {
            for (uint256 i = 0; i < users.length; ++i) {
                if (tokens[users[i]].exists) continue; 

                _mint(false, users[i]);
                _initializeMarkets(users[i]);
                delete stakedAt[users[i]];
            }
        }
    }
```



### [Low-02]: Scores#calculateScore() doing unecessary computation for `alpha=0` and `alpha=1` 

The calculate score uses the formula `xvs^𝝰 * capital^(1-𝝰)`, where the alpha lies b/w (0,1). Its also possible for the value of alpha to be either 0 or 1. However, if it so, the calculateScore performing unnecesary logarithmic and exponentials calculation, increasing the complexity and adding more attack surfaces, which is definitely not needed. 

As looking at the above formula, we can say
for 𝝰=0, the score is corresponding to `capital` and
for 𝝰=1, the score eq staked `xvs`

This value can be directly returns.

**Recommendation**
*File: Scores.sol*
```solidity
    function calculateScore(
        uint256 xvs,
        uint256 capital,
        uint256 alphaNumerator,
        uint256 alphaDenominator
    ) internal pure returns (uint256) {
   // ...SNIP...
        if (xvs == capital) return xvs;

+      if(alphaNumerator == 0) return capital;
+      if(alphaNumerator == alphaDenominator) return xvs; 

        bool xvsLessThanCapital = xvs < capital;

        // (xvs / capital) or (capital / xvs), always in range (0, 1)
        int256 ratio = xvsLessThanCapital ? FixedMath.toFixed(xvs, capital) : FixedMath.toFixed(capital, xvs);

        // e ^ ( ln(ratio) * 𝝰 )
        int256 exponentiation = FixedMath.exp(
            (FixedMath.ln(ratio) * alphaNumerator.toInt256()) / alphaDenominator.toInt256()
        );

        if (xvsLessThanCapital) {
            // capital * e ^ (𝝰 * ln(xvs / capital))
            return FixedMath.uintMul(capital, exponentiation);
        }

        // capital / e ^ (𝝰 * ln(capital / xvs))
        return FixedMath.uintDiv(capital, exponentiation);
    }
```
### [Low-03]: `_claimInterest` set the user rewardIndex to non-zero, when its already set to 0 during `_burn` call

The `burn` is restricted call, can be called to burn prime token. The function internally calls `_burn` which first accrue rewards for the user and then set the user `score` and `rewardIndex` to zero. 


```solidity
    function _interestAccrued(address vToken, address user) internal view returns (uint256) {
        uint256 index = markets[vToken].rewardIndex - interests[vToken][user].rewardIndex;
        uint256 score = interests[vToken][user].score;

        return (index * score) / EXP_SCALE;
    }
```
Aftwards the `burn` call, user still can claim her collected rewards via `_claimInterest` function. However, this call changes the user rewardIndex from 0 to a non-zero market rewardIndex, which could open unexpected attack paths. 

**Recommendation**
*File: Prime.sol*
```solidity
    function _claimInterest(address vToken, address user) internal returns (uint256) {
        uint256 amount = getInterestAccrued(vToken, user);
        amount += interests[vToken][user].accrued;

        if(tokens[user].exists) interests[vToken][user].rewardIndex = markets[vToken].rewardIndex;
        interests[vToken][user].accrued = 0;
        // ...SNIP...
    }
```

### [Low-04]: Revocable token issued by owner can be immediately `burn`, reducing the `pendingScoreUpdates` by 1

The `issue` function directly issue prime tokens to users. It do not check the user eligibility, when minting the revocable tokens. If the `xvsUpdated` function which has a public visibility is being called for that user, it eventually `_burn` the revocable tokens of that users. 

```solidity
    function xvsUpdated(address user) external {
        uint256 totalStaked = _xvsBalanceOfUser(user);
        bool isAccountEligible = isEligible(totalStaked);

        if (tokens[user].exists && !isAccountEligible) {
            if (tokens[user].isIrrevocable) {
                _accrueInterestAndUpdateScore(user);
            } else {
                _burn(user);  // user hold revocable token, but not eligible
            }
        } else if (!isAccountEligible && !tokens[user].exists && stakedAt[user] > 0) {
        // ...SNIP...
    }

```

However, the problem is `_burn` internally calls `_updateRoundAfterTokenBurned`, which reduces the `pendingScoreUpdates` count by 1. We knew already that, the `pendingScoreUpdates` never actually been increment or updated when the revocable token issued. 

```solidity
    function _updateRoundAfterTokenBurned(address user) internal {
        if (totalScoreUpdatesRequired > 0) totalScoreUpdatesRequired--;

        if (pendingScoreUpdates > 0 && !isScoreUpdated[nextScoreUpdateRoundId][user]) {
            pendingScoreUpdates--;
        }
    }
```

This could lead to an unexpected behaviour at the places where it suppose to be higher than the current value.


