
## Low

[L - 1] `Prime` - Round id is only considered on [`updateScores()`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200), not in other functions.
The function `updateScores()` checks the variables `pendingScoreUpdates` and `nextScoreUpdateRoundId` before allowing for updates to happen, and after updates `pendingScoreUpdates` and `isScoreUpdated`. These conditions can be easily circumvented by calling the function [`accrueInterestAndUpdateScore()`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L389) that allows for updates to happen without going through any of the previously mentioned conditions.

[L - 2] `Prime` - `updateScores()` [reverts](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L207) if a user has exited before the call.
If a user no longer exists before the call to `updateScores()` has been made, the whole transaction will revert and will not allow for the other user's scores to be updated, incurring in extra gas costs by having to call this function again with a new input.

[L - 3] There is no input validation for the [`updateMultiplier`][https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L263] function
In this function, neither the variables `supplyMultiplier` nor `borrowMultiplier` are checked to be between reasonable values


## Non-Critical Issues

[N - 1] `PrimeLiquidityProvider` Change function name [`_ensureZeroAddress`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L344) -> `_ensureValidAddress`
The function `_ensureZeroAddress` does not ensure that the address given is a zero address, like the name suggests, it ensures that it is a valid address (i.e. not a zero address), and thus it should be named `_ensureValidAddress`

[N - 2] `PrimeLiquidityProvider`, [`lastAccruedBlock`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L24)  Incorrect comment  
The comment "/// @notice The rate at which token is distributed to the Prime contract" does not accurately describe the variable. A more accurate comment could be "The block number of the latest accrual for that token"

[N - 3] `Prime` - There is no guarantee that when a new round starts, all users have updated their scores. 
There are several functions ([`updateAlpha`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L254),  [`updateMultipliers`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L279) and [`addMarket`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L304C9-L304C34)) that call the function `_startScoreUpdateRound` , this function updates the values `nextScoreUpdateRoundId`, `totalScoreUpdatesRequired`, and `pendingScoreUpdates`. However, there is no check to ensure that the previous round of updates has ended and therefore there are still `pendingScoreUpdates` for that specific round.