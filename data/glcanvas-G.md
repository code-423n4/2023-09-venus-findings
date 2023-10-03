# [G-01] Move token's struct filling after checks
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L704-L720

Move 
```solidity
        tokens[user].exists = true;
        tokens[user].isIrrevocable = isIrrevocable;
```
after 
```solidity
if (totalIrrevocable > irrevocableLimit || totalRevocable > revocableLimit) revert InvalidLimit();
```
To save gas if limit will be exhausted.