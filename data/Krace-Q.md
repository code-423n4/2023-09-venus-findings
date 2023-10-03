### [Low Risk](#low-risk-1)

| Total Low Risk Issues | 3 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [L-01](#l-01-the-accruetokens-can-be-abused-to-reduce-the-cumulative-number-of-tokens) | The accrueTokens can be abused to reduce the cumulative number of tokens | 1 |
| [L-02](#l-02-dos-in-accruetokens-if-owner-wrongly-sweeptoken) | Dos in accrueTokens if owner wrongly sweepToken | 1 |
| [L-03](#l-03-underflow-in-_calculatescore) | Underflow in `_calculateScore` | 1 |

### [Non-Critical](#non-critical-1)

| Total Non-Critical Issues | 1 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [NC-01](#nc-01-functions-should-be-a-modifier) | Functions should be a modifier | 2 |


## Low Risk

### [L-01] The accrueTokens can be abused to reduce the cumulative number of tokens

[`accrueTokens`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249-L272) is used to accumulate the amount of tokens across the block, and it is a public function that could be invoked by anyone. 
However, if the balance of `this` is less than the accumulated value of the token during the period, the token can only obtain the difference between `this` balance and the previously accumulated value. Therefore, a malicious attacker can call this function before the contract has sufficient balance, preventing the token from obtaining the expected accumulated value.

```
    uint256 balanceDiff = balance - tokenAmountAccrued[token_];
    if (distributionSpeed > 0 && balanceDiff > 0) {
        uint256 accruedSinceUpdate = deltaBlocks * distributionSpeed;
        uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);
```


### [L-02] Dos in accrueTokens if owner wrongly sweepToken

Function [`sweepToken`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225) allows the owner to sweep any tokens from this contract, including the initialzed tokens.

```
    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
        uint256 balance = token_.balanceOf(address(this));
        if (amount_ > balance) {
            revert InsufficientBalance(amount_, balance);
        }

        emit SweepToken(address(token_), to_, amount_);

        token_.safeTransfer(to_, amount_);
    }
```

If the owner accidentally sweeps some of the accumulated tokens, the [`accrueToken`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249-L272) function might encounter an underflow when calculating the accumulated tokens because the balance at that moment may not be sufficient to cover the originally accumulated tokens. This could lead to temporary denial of service.

```
    uint256 distributionSpeed = tokenDistributionSpeeds[token_];
    uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
//@audit If balance of this is less than the tokenAmountAccrued due to `sweepToken`, underflow occurs
    uint256 balanceDiff = balance - tokenAmountAccrued[token_];
```



### [L-03] Underflow in `_calculateScore`

If the decimal of `vToken` is larger than 18, then [`_calculateScore`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L661) will always underflow.

```
    function _calculateScore(address market, address user) internal returns (uint256) {
        uint256 xvsBalanceForScore = _xvsBalanceForScore(_xvsBalanceOfUser(user));

        IVToken vToken = IVToken(market);
        uint256 borrow = vToken.borrowBalanceStored(user);
        uint256 exchangeRate = vToken.exchangeRateStored();
        uint256 balanceOfAccount = vToken.balanceOf(user);
        uint256 supply = (exchangeRate * balanceOfAccount) / EXP_SCALE;

        address xvsToken = IXVSVault(xvsVault).xvsAddress();
        oracle.updateAssetPrice(xvsToken);
        oracle.updatePrice(market);

        (uint256 capital, , ) = _capitalForScore(xvsBalanceForScore, borrow, supply, market);
        //@audit if vTOken.decimals() is less than 18, underflow occurs
        capital = capital * (10 ** (18 - vToken.decimals()));

        return Scores.calculateScore(xvsBalanceForScore, capital, alphaNumerator, alphaDenominator);
    }
```

## Non-critical

### [NC-01] Functions should be a modifier

- PrimeLiquidityProvider.sol ( [#L332](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L332) , [#L344](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L344) ):

These two functions should be changed to a modifier.

```
    function _ensureTokenInitialized(address token_) internal view {
        uint256 lastBlockAccrued = lastAccruedBlock[token_];

        if (lastBlockAccrued == 0) {
            revert TokenNotInitialized(token_);
        }
    }
```

```
    function _ensureZeroAddress(address address_) internal pure {
        if (address_ == address(0)) {
            revert InvalidArguments();
        }
    }
```

