[G-1] pendingScoreUpdates can be updated at the end of updateScores instead of using `pendingScoreUpdates--` at every loop.

```solidity
    function updateScores(address[] memory users) external {
        if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
        if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

        for (uint256 i = 0; i < users.length; ) {
            address user = users[i];

            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

            address[] storage _allMarkets = allMarkets;
            for (uint256 j = 0; j < _allMarkets.length; ) {
                address market = _allMarkets[j];
                _executeBoost(user, market);
                _updateScore(user, market);

                unchecked {
                    j++;
                }
            }

            pendingScoreUpdates--;
            isScoreUpdated[nextScoreUpdateRoundId][user] = true;

            unchecked {
                i++;
            }

            emit UserScoreUpdated(user);
        }
    }
```

## Recommendation
move `pendingScoreUpdates--` outside of the loop and replace it with 

```solidity
pendingScoreUpdates -= users.length
```

