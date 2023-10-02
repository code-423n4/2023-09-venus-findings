## 1. Approach taken in evaluating the codebase
- Read the documentation about Prime tokens first before looking at the contracts (Prime.sol, PrimeLiquidityProvider.sol)
- Studied the Storage variables in PrimeStorage.sol
- Looked at other important contracts that were out of scope but was necessary to understand the codebase, such as XVSVault.sol and Oracle.sol

## 2. Mechanism summary and review (Prime.sol)

### Getting a Prime Token  
- User deposits 1000 XVS into the XVS vault 
- In the first deposit, `xvsUpdated()` will be called and it sets `stakedAt[user] = block.timestamp`
- User waits for 90 days. During this period, if the XVS staked goes below 1000, the staked time will reset to zero
- After 90 days, the user can call `claim()` to get a Prime token
- When `claim()` is called, a Prime token is minted and the score is updated for the user.
- The user must make sure that 1000 XVS is deposited at all time, otherwise the Prime token will be burned and the user has to repeat the process to get another Prime Token

##### Review
- `xvsUpdated()` is publicly accessible, so anyone can call update for anyone else. Not sure of the repercussions, but one thing that I noticed was that anyone can immediately burn someone else's Prime token if they do not have >1000 XVS staked and was gifted a token by the admin

### Admin issuing an Prime Token (Irrevocable)
- Admin calls `issue()` to give either a revocable prime token or an irrevocable prime token to the user
- If the user already has a revocable token, then the current revocable token that the user has will be upgraded to an irrevocable one
- If the user does not have any tokens, the user will immediately get an irrevocable one
- Total number of revocable and irrevocable tokens will be updated accordingly

### Admin issuing an Prime Token (Revocable)
- If the user is currently waiting for 90 days for a revocable token, then the user will immediately get a revocable token and the staking time will reset to zero.
- If the user does not have a revocable token and has not deposited any funds, then the user will still get a revocable token and the staking time will reset to zero

##### Review
- Staking duration and amount is not checked when minting

### Accrual of Interest by User
- Assumes that user already has a Prime token
- User deposits more XVS, triggering `xvsUpdated()`
- `xvsUpdated()` checks whether user is still eligible (>1k XVS) and calls `_accrueInterestAndUpdateScore()`
-  `_accrueInterestAndUpdateScore()` calls `_executeBoost()` and `_updateScore()`
- `_executeBoost` calls `accrueInterest()` which updates the rewardIndex of the market and calls `_interestAccrued() which updates the amount of accrued interest of the user. The user's `rewardIndex` is then set to the market's `rewardIndex`.
- _updateScore() calculates the new score of the user and adds it to the `sumOfMembersScore`, and updates the user's score

##### Review
- If `xvsUpdated()` is called within the same block (such as calling deposit in XVSVault and then calling `claimInterest()`, the function will still run but the market `rewardIndex` will stay the same (delta = 0). Could be possible to add a revert if there was no interest accrued by the user
- RewardIndex of the market always increases if there is delta, and never decreases. Could be possible that the value of `rewardIndex` from the market will reach uint256 max amounts and result in revert. 

### Admin actions (updateAlpha, updateMultiplier, addMarket)
- When either of the three functions are called, `_startScoreUpdateRound()` is called
- `_startScoreUpdateRound()` will update `nextScoreUpdateRoundId`, `totalScoreUpdatesRequired` and `pendingScoreUpdates`
- After `_startScoreUpdateRound()` is called, `updateScores()` must be called immediately after so that there is not much discrepancy between the new market score and the old market score 
- `updateScores()` will check if the user has a token and if the score was already updated. It then subtracts pendingScoreUpdates by 1 for every user updated in round denoted by the `nextScoreUpdateRoundId` 

##### Review
- If `_startScoreUpdateRound()` is in between `updateScores()`, the `pendingScoreUpdates` variable may be counted wrongly, depending on how the admin does the order of transactions (more details found in submitted medium issue)

## 3. Centralization risks 
- Admin is in charge of pausing the claiming of interest; a malicious admin can pause indefinitely
- Admin is in charge of multipliers and alpha numbers. In `updateAlpha()` there seems to be limitations on the alpha value via `_checkAlphaArguments()`, but there is no limitations on the multiplier value in `updateMultipliers()`. Would be good to add some upperbounds to the updateMultipliers section.
- Admin is in charge of setting the upper limit of irrevocable and revocable tokens. 

## 4. Architecture recommendations
- Looks like the values in PrimeStorage.sol are unchangeable, which on one hand provides security (lesser centralization risk) but on the other hand demotivates flexibility (staking period cannot be changed etc). 
##### Examples of inflexibility
- `STAKING_PERIOD` is fixed at 90 days forever, which may be too strict
- ` MINIMUM_STAKED_XVS` is fixed at 1000, which may be impossible to reach if the token is worth a hefty price. Eg if 1 XVS is worth 1000 dollars, then a user must have 1 million just to get a Prime token.
- Similarly, `MAXIMUM_XVS_CAP` is fixed at 100000, which may be easy to reach if the token is not worth alot. Eg if 1 XVS is worth 0.00001, then only a dollar is needed to reach maximum cap 
- From the looks of it, the numbers cannot be changed unless a new contract is introduced. An overhaul of a new PrimeStorage version 2 seems really tedious to do, so it will be more flexible to allow the admin to change the important variables with lower and upper limits.

### Time spent:
12 hours