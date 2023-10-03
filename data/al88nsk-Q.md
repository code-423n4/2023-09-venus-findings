# L-1 Function updateScores spends all gas and reverts if a user has score updated

## Summary

Function `updateScores` incorrectly handles case when a user's score is already updated.

## Vulnerability Details

There is a `for` loop in the function `updateScores` that updates score for every user in `users`. If a user's score is already updated, the user should be skipped and the next user should be updated. For this case there is an `if` statement (https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L208C1-L208C1):

```solidity
 if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;
```

However, here is an issue. The variable `i` that is responsible for the current user in the loop is incremented in the end of the loop, but due to the `continue` statement, this code is not reached, the value of `i` remains and the current user is processed again and again until all the gas is spent, and then the function reverts.

## Recommended Mitigation Steps

When user's score is updated, the `i` value must be incremented.

# L-2 Function getPendingInterests does not correctly handles VBNB market

## Summary

Function `getPendingInterests` returns incorrect underlying token for VBNB market.

## Vulnerability Details

`VBNB` market is specifically handled in function `_getUnderlying` (https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L930-L936):

```
function _getUnderlying(address vToken) internal view returns (address) {
    if (vToken == VBNB) {
        return WBNB;
    } else {
        return IVToken(vToken).underlying();
    }
}
```
So, when market is `VBNB`, the underlying token must be `WBNB`, otherwise the `underlying()` function result must be used. However, the function `getPendingInterests` uses `IVToken(vToken).underlying()` directly for all markets and has not special handling for `VBNB` market (https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L183-L186):

```solidity
pendingInterests[i] = PendingInterest({
    market: IVToken(market).underlying(),
    amount: interestAccrued + accrued
});
```

So, the VBNB market is not handled correctly, `_getUnderlying` must be used instead.

## Recommended Mitigation Steps

Use function `_getUnderlying` to get underlying token in function `getPendingInterests`.