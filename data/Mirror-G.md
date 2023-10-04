## [G-01] Initialized with default value

The default value for variables is zero, so no need to initialize them to zero.

*1 instance*

[Prime.sol#156](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L156)
```js
function initialize(
    ...
) external virtual initializer {
    ...
    nextScoreUpdateRoundId = 0;
    ...
}
```

## [G-02] Optimize validation of `accrueTokens()`

Only non-zero address token can be initialized. Thus, `_ensureZeroAddress` check in the `accrueTokens()` function could be removed.

*1 instance*

[Prime.sol#250](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L250)

```diff
function accrueTokens(address token_) public {
-    _ensureZeroAddress(token_);
    _ensureTokenInitialized(token_);
    ...
}

function _initializeToken(address token_) internal {
    _ensureZeroAddress(token_);
    ...
} 
```