| Identifier  | Issue                                                                                 |
|-------------|---------------------------------------------------------------------------------------|
| L-01        | Consider using `continue` instead of reverting in `Prime::updateScores()`. |
| L-02        | Consider limiting `_alphaDenominator` max value.                                |
| QA-01       | Fix `UpdatedAssetsState` NatSpec                                 |
| QA-02       |  Fix `UpdatedAssetsState` NatSpec.                                                                                     |
| QA-03       |     Typos                                                                                  |


## L-01 Consider using `continue` instead of reverting in `Prime::updateScores()`.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L200-L230

Reverting when user doesn't hold Prime tokens may open possibilities for malicious users to DoS `Prime::updateScores()`. To illustrate this point, consider the following scenario: a malicious user front-runs the `updateScores` transaction, unstakes his XVS tokens, resulting in his token being burned, and subsequently causing `updateScores` to revert. To mitigate this risk, consider using the `continue` statement to skip the specific loop iteration for users who currently do not possess Prime tokens, rather than reverting, like show below.

```diff
     function updateScores(address[] memory users) external {
         if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
         if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

         for (uint256 i = 0; i < users.length; ) {
             address user = users[i];

-            if (!tokens[user].exists) revert UserHasNoPrimeToken();
+            if (!tokens[user].exists) continue;
             if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;
```


## L-02 Consider limiting `_alphaDenominator` max value.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L809-L813

`_checkAlphaArguments` verifies newly set alpha parameters. However it only checks if `_alphaDenominator == 0` and `_alphaNumerator > _alphaDenominator`. Setting `_alphaNumerator` or `_alphaDenominator` high enough can make `Scores::calculateScore()` always revert due to the conversion to `int256`. Consider checking that `_alphaDenominator <= type(int256).max`


## QA-01 Fix `UpdatedAssetsState` NatSpec.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L56-L57

```diff
-    /// @notice Emitted asset state is update by protocol share reserve
+    /// @notice Emitted when asset state is update by protocol share reserve
    event UpdatedAssetsState(address indexed comptroller, address indexed asset);
```

## QA-02 Fix `getAllMarkets()` NatSpec.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L467

```diff
    /**
     * @notice Retrieves an array of all available markets
-    * @return an array of addresses representing all available markets
+    * @return allMarkets an array of addresses representing all available markets
     */
    function getAllMarkets() external view returns (address[] memory) {
        return allMarkets;
    }
```

## QA-03 Typos.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L385

```diff
-    * @notice accrues interes and updates score for an user for a specific market
+    * @notice accrues interest and updates score for an user for a specific market
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L604

```diff
-     * @notice accrues interes and updates score of all markets for an user
+     * @notice accrues interest and updates score of all markets for an user
```

