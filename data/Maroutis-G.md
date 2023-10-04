Link https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

The conditions:
```
if (!tokens[user].exists) and 
if (!markets[market].exists)
```
Are repeated many times in the Prime.sol contract. Especially when the functions `_executeBoost` and `_updateScore` are called (for example : `_accrueInterestAndUpdateScore` and `updateScores`). 

- It would be best to optimize these functions so that the same conditions are not being called a second time.