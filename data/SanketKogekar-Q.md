- The `setPrimeToken` method is not set back to the same value, and avoids unnecessary emit. 

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177-L182

- The protocol should revert `releaseFunds()` if accruedAmount == 0 instead of transfering 0 amount and emitting token transfer event.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192-L205

- In the `toFixed` function, there is a check for d.toInt256() < n.toInt256(), but it does not handle the case where d is 0. This can result in a division by zero error.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/libs/FixedMath.sol#L22-L26

- In `uintMul` and `uintDiv`, consider using `<= 0` in comparison to avoid division by 0.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/libs/FixedMath.sol#L34-L38

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/libs/FixedMath.sol#L46-L50

- The `calculateScore()` function uses the FixedMath library to perform fixed-point arithmetic. However, the `FixedMath.uintMul()` and FixedMath.uintDiv() functions can overflow if the inputs are too large. This might allow an attacker to steal funds by manipulating the xvs, capital, alphaNumerator, or alphaDenominator parameters.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/libs/Scores.sol#L22-L23

- This varaible is redeclared:

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L176

Change it simply to: `pendingInterests = new PendingInterest[](_allMarkets.length);`