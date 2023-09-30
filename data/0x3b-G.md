| *Issue* | *Description*                                                                                                                                                            |
|---------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [G-01]  | In [updateScores](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230) you don't need to accrue every market for every user |
| [G-02]  | No need to save [totalIncomePerBlockFromMarket](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L971)                             |

### [G-01] In [updateScores](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230) you don't need to accrue every market for every user
In [updateScores](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230) the internal [_executeBoost](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L779-L787) is called where for every user every market is accrued. However this is not necessary since every market needs to be accrued once. This will save a lot of gas since for a transaction with 20 users, all markets are gonna be accrued 20 times, while only 1 accrue is needed per market. 

I would suggest to first [accrue](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L211-L219) all of the markets, and then add and sync it with [_interestAccrued](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L918-L923). To do this you will need to add a for loop to [updateScores](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230) to accrue all markets and then you can modify [_executeBoost](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L779-L787) to not accrue any market.

```diff
    function _executeBoost(address user, address vToken) internal {
        if (!markets[vToken].exists || !tokens[user].exists) {return;}

-       accrueInterest(vToken);
        interests[vToken][user].accrued += _interestAccrued(vToken, user);
        interests[vToken][user].rewardIndex = markets[vToken].rewardIndex;
    }
```

### [G-02] No need to save [totalIncomePerBlockFromMarket](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L971)
Under [_incomeDistributionYearly](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L970-L979) `totalIncomePerBlockFromMarket` is saved and used in one place, there is no need for such variable and you can insert the function in the math logic. Same can be said for `incomePerBlockForDistributionFromMarket`, as again it is used only in `amount = BLOCKS_PER_YEAR * incomePerBlockForDistributionFromMarket;`

```diff
-       uint256 totalIncomePerBlockFromMarket = _incomePerBlock(vToken);
-       uint256 incomePerBlockForDistributionFromMarket = (totalIncomePerBlockFromMarket * _distributionPercentage()) / IProtocolShareReserve(protocolShareReserve).MAX_PERCENT(); 

+       uint256 incomePerBlockForDistributionFromMarket = (_incomePerBlock(vToken) * _distributionPercentage()) / IProtocolShareReserve(protocolShareReserve).MAX_PERCENT(); 
```
