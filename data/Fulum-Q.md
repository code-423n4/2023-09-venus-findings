# 01 `Prime::updateScores()` can be reverted if a user changes status to `!tokens[user].exists`.

On the `Prime::updateScores()` executed by your script to update the scores of all users when an update of aplha values or multipliers values are made, you have this line: 

```solidity
    function updateScores(address[] memory users) external {
       ...
            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            ...
    }
```

This line revert the function if one user in the batch doesn't exist, make all the updates in the batch revert.
I don't know what is exactly the process of the script, but you say you take all the users existing in the block who the updates of alpha or multipliers are made to make the update.
But if a user exist in the list and he is revoked in the block just after but before the execution of the script, the user is send in `Prime::updateScores()` but he doesn't exist and trigger the line of the function, and the function revert. If you intend the script work well, the scores of a part of users are not updated well and the offset of the `sumOfMembersScore` are not well updated.

## Recommendations: 

Simply use the `continue` instead of `revert` to let the function continue if a user no longer exists:

    -- if (!tokens[user].exists) revert UserHasNoPrimeToken();
    ++ if (!tokens[user].exists) continue;
