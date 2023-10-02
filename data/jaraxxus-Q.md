### Table of Contents

- [L-01] All scores should be updated first before the market resumes its operation
- [L-02] Scores can be updated twice
- [L-03] rewardIndex may reach overflow, resulting in revert
- [I-01] The `isIrrevocable` variable in the Token struct could be a uint instead of a boolean 
- [I-02] Markets cannot be removed 
- [I-03] There is no cap for multipliers


### [L-01] All scores should be updated first before the market resumes its operation

Changing the multiplier / alpha may created a huge shift in the APR percentage. If updateScore() is not called immediately, the user can earn a higher/lower APR than intended. A mitigation step is to make sure that pendingScoreUpdates is 0 before the market continues to operate.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L200-L203

### [L-02] Scores can be updated twice

If user deposits/withdraw or change borrow/supply values after alpha/multiplier is changed but before updateScore() is changed, then the user would have updated the score twice. A mititgation step is that isScoreUpdated[nextScoreUpdateRoundId][user] is set to true in `_updateScore()` internal function instead of updateScores() external function.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L222

### [L-03] rewardIndex may reach overflow, resulting in revert

Value of markets[vToken].rewardIndex is ever increasing. Everytime accrueInterest() is called, if delta is more than 0, rewardIndex will increase. Since there is no reset for this value, it may eventually reach uint256 max value and when that happens, the protocol will not work anymore because accrueInterest() will always revert.

```
        if (markets[vToken].sumOfMembersScore > 0) {
            delta = ((distributionIncome * EXP_SCALE) / markets[vToken].sumOfMembersScore);
        }

        markets[vToken].rewardIndex = markets[vToken].rewardIndex + delta;
```

Consider the case of reducing the rewardIndex of the market.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L584

### [I-01] The `isIrrevocable` variable in the Token struct could be a uint instead of a boolean

When a token is burned, the isIrrevocable boolean is set to false 

```
        tokens[user].exists = false;
        tokens[user].isIrrevocable = false;
```

This might be misintepreted as setting the token to isIrrevocable means that it is a revocable token. It does not mean that it is a null token. It may be better to set the variable to 0,1,2 values, where 0 means that the token does not exist, 1 means that the token is revocable and 2 means that the token is irrevocable. This way, the struct can be condensed to just 1 variable as well.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L7-L10

### [I-02] Markets cannot be removed 

There is addMarket functionality but no remove market functionality. In the event that a token is compromised, users should not be able to earn any APR / supply/borrow in that market.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L288-L290

### [I-03] There is no cap for multipliers

Admin can increase the multiplier of a market but there is no maximum cap. Admin can increase up to 100000x, which may happen if admin is compromised. This may affect users if they are borrowing from the market and their borrowMultiplier increases unexpectedly. Good to set a cap, maybe up to 20x. 

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L263-L265