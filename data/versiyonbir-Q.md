# Imprecise Arithmetic Operations Order

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249C2-L272C6

There's a potential issue with the order of arithmetic operations in the _setTokenDistributionSpeed function. Solidity uses fixed-point arithmetic, and the order of operations can impact precision. In particular, this line:
```solidity
uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate); 
```
It's performing a subtraction (balance - tokenAmountAccrued) and then a comparison (<=). Depending on the scale of the numbers involved, this could lead to imprecise results. Consider revising the order of operations to ensure precision.

# Dangerous Unary Expression

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol

The "dangerous unary expression" vulnerability warning is related to the use of the .toInt256() method on variables of type uint256 and int256 without proper checks for potential overflows or underflows. Here's the specific code that triggers this warning:
```solidity
if (d.toInt256() < n.toInt256()) revert InvalidFraction(n, d);
// ...
return uint256((u.toInt256() * FixedMath0x.FIXED_1) / f);
```
In Solidity, when converting a large uint256 to an int256, there's a risk of overflow if the uint256 value is greater than int256's maximum value. Similarly, there's a risk of underflow if converting a negative int256 to a uint256. These potential issues are what the warning is pointing out.
To address this vulnerability, you should add checks to ensure that the conversion from uint256 to int256 and vice versa won't result in overflow or underflow. You can use conditional statements to check for this before performing the conversion. Here's an example of how you can modify the code:
```solidity
function toFixed(uint256 n, uint256 d) internal pure returns (int256) {
    if (d == 0) {
        revert InvalidFraction(n, d); // Avoid division by zero
    }

    int256 nInt = int256(n);
    int256 dInt = int256(d);

    if (dInt < nInt) {
        revert InvalidFraction(n, d);
    }

    return (nInt * FixedMath0x.FIXED_1) / dInt;
}

function uintDiv(uint256 u, int256 f) internal pure returns (uint256) {
    if (f < 0) {
        revert InvalidFixedPoint();
    }
    
    int256 uInt = int256(u);

    if (uInt < 0) {
        revert InvalidFixedPoint();
    }

    // Multiply `u` by FIXED_1 to cancel out the built-in FIXED_1 in f
    return uint256((uInt * FixedMath0x.FIXED_1) / f);
}
```
In the modified code, we've added checks to ensure that the conversions between uint256 and int256 are safe. Additionally, we've added a check to prevent division by zero when d is zero. These changes help mitigate the "dangerous unary expression" vulnerability.

