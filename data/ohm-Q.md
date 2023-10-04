## Adding Zero Address Checks to Critical Functions

Input validation should be performed for critical functions. In the sweepToken function, the 'to' address must be checked for the zero address before transferring the tokens.

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216

## Unnecessary Type casting

In the toFixed() function of FixedMath.sol, you can simplify int256(d.toInt256()) to just d.toInt256() because the toInt() function already returns the integer value. There is no need for type casting.

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L25

```solidity
    function toInt256(uint256 value) internal pure returns (int256) {
        // Note: Unsafe cast below is okay because `type(int256).max` is guaranteed to be positive
        require(value <= uint256(type(int256).max), "SafeCast: value doesn't fit in an int256");
        return int256(value);
    }
```

