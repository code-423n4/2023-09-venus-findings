## [L-01] `Prime.updateScores` can be griefed if a User frontruns the transaction and burns his Prime token.

[Link to the code](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L207)

`updateScores` function allows anyone to update a user's scores once if `_startScoreUpdateRound()` is called. This function reverts if a user does not exist.

### Proof of Concept
Let's say admin wants to update scores for 100 users.
Admin calls the `Prime.updateScores` and the transaction goes to mempool.
At the same time, one of the users decides that he wants to burn his Prime Token and withdraws XVS from the `XVSVault`. 
Users transaction is completed first, meaning that now in the 100 user list there is one user, who does not exist anymore and the transaction will revert.

```solidity
            address user = users[i];

            // @audit-issue Skipping here is preffered.
            // Because if there are 100 users and 1 user is invalid, then the whole transaction will revert
            // For example: Venus Devs call updateScores for 100 users, but one user burns his Prime token -> tx reverts
            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

```

### Recommended Solution
Instead of reverting if a user does not exist, skip.
```solidity
            if (!tokens[user].exists || isScoreUpdated[nextScoreUpdateRoundId][user]) continue;
```
