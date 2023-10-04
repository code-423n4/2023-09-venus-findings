
## Merge statments to prevent using state vairable multiple time on same function

In [`_mint()`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L704)  and [`_upgrade()`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L762) function
```diff
function _mint(bool isIrrevocable, address user) internal {

if (tokens[user].exists) revert IneligibleToClaim();

  

tokens[user].exists = true;

tokens[user].isIrrevocable = isIrrevocable;

-- if (isIrrevocable) {

-- totalIrrevocable++;  

++ if (isIrrevocable && ++totalIrrevocable>irrevocableLimit) {

++ revert InvalidLimit();

} else {
-- totalRevocable++;// @audit gas saving merget below if statment

++ if(++totalRevocable>revocableLimit) revert InvalidLimit();

}

emit Mint(user, isIrrevocable);

}
```

```diff
function _upgrade(address user) internal {

Token storage userToken = tokens[user];

  

userToken.isIrrevocable = true;
-- totalIrrevocable++;
totalRevocable--;

  
-- if (totalIrrevocable > irrevocableLimit) revert InvalidLimit();
++ if (++totalIrrevocable > irrevocableLimit) revert InvalidLimit();

  

emit TokenUpgraded(user);

}
```
## Use `msg.sender` instead of reading state twice in [`releaseFunds()`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192)
```diff
function releaseFunds(address token_) external {

if (msg.sender != prime) revert InvalidCaller();

if (paused()) {

revert FundsTransferIsPaused();

}

  

accrueTokens(token_);

uint256 accruedAmount = tokenAmountAccrued[token_];

tokenAmountAccrued[token_] = 0;

  

emit TokenTransferredToPrime(token_, accruedAmount);

  

-- IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);
++ IERC20Upgradeable(token_).safeTransfer(msg.sender, accruedAmount);

}
```

## Avoid using getBlockNumber()
As it can be obtained with `block.number`
2 instance
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L254
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L288
```

