# Imprecise Arithmetic Operations Order

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249C2-L272C6

There's a potential issue with the order of arithmetic operations in the _setTokenDistributionSpeed function. Solidity uses fixed-point arithmetic, and the order of operations can impact precision. In particular, this line:
```solidity
uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate); 
```
It's performing a subtraction (balance - tokenAmountAccrued) and then a comparison (<=). Depending on the scale of the numbers involved, this could lead to imprecise results. Consider revising the order of operations to ensure precision.

