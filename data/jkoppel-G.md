* Reduce redundant calls to `accrueInterest`

`accrueInterest` is called a lot. Many of these calls are redundant. For instance, in `updateScores`:

```solidity
 for (uint256 i = 0; i < users.length; ) {
    // ...
    for (uint256 j = 0; j < _allMarkets.length; ) {
      address market = _allMarkets[j];
      _executeBoost(user, market);
      // ...
    }
    // ...
```

This loop calls `_executeBoost` on the same market once per user. `_executeBoost` in turn calls `accrueInterest` on that market. These calls are redundant.

* Mark all privileged external functions as `payable`

Reason: Solidity compiler emits extra code to check that `value == 0`. Privileged functions can be assumed not to accidentally be called with native token, making this unnecessary