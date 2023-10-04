# Any comments for the judge to contextualize your findings

Some of the submissions were sent, keeping in mind the blockchains the protocol intends to deploy. The Binance Smart Chain is the main deployment chain and the one we can be assured of. Whether the sponsors have serious plans to implement the code as is to other blockchains or not is beyond the scope of the auditor and was assumed to be true for the listed chains in the contest's page description.

# Approach taken in evaluating the codebase

A more systematic approach was taken for this audit, as opposed to what I usually go through. The following list includes the approach taken for the audit.

- First and foremost, read the documentation 
- Check every single variable, especially globals, separately from their function (e.g., study what it does, and then check logic).
- Cross-reference variables within a contract and write down what other variables they can affect.
- Check function call stack separately, preferably together with tests
- Check if something critical in the project gets deprecated by an EIP 
- Check for vulnerabilities at a higher abstraction level that require external context 
- Check test coverage.
- Check internal flow of function executions 
- Check what chain it is using 
- Check the flow of money at the end
- If a pattern that led to a vulnerability was found, check for the same/similar patterns in the rest of the code.

# Mechanism Review

There's a pause function in the LP, which is properly used according to what the documentation states.

`xvsUpdated()` is a function that can be freely called by anyone. In case a user, for some reason, changes his position in a way that would decrease their eligibility to below minimum, anyone can call this and burn below minimum positions. 

Users below the minimum threshold can be minted irrevocable tokens. This is a design choice of the protocol, as people with zero tokens can have prime tokens too

#### Global variables
In Prime.sol, there are two important data structures, the markets and interests mapping structs. The interests map a vtoken to the user and to the user's indexes, which include his score, accrued value and reward index. This array is set in the functions _initializeMarkets, _claimInterest, _burn, _executeBoost and _updateScore.

The markets arrays are set in the functions updateMultipliers, addMarket, accrueInterest, _initializeMarkets, _burn, _updateScore. The market index is set at 0 on addMarket, and it's incremented by `delta` on accrued interest, which means a market index can only increase. The interests index are set as equal to markets index on initialize, claimInterest and executeBoost. It's set as 0 on token burn.

The sumOfMembersScore is set as 0 on addMarket, added interests score on initializeMarkets, reduced score on burn, modified on updateScore. The score is set in initializeMarkets, set to 0 in burn or _updateScore.

We can conclude that these arrays are working as intended.

#### Functions:
##### In Prime.sol:
updateScores() can update the score of users who just joined through initializeMarkets. It requires the totalScoreUpdatesRequired variable in order to function, which is set to total irrevocable + total revocable tokens. Users who just claimed prime tokens increase totalRevocable or totalIrrevocable amount.

accrueInterest(): This function accrues tokens from the PSR and the PLP, if the contract doesn't have enough funds. When this is done, these contracts will send all accrued tokens and set the unreleased amount variable to zero.

burn(): Removes the pending score update of a user and sets their score to zero

_capitalForScore(): cap may be susceptible to MEV. Even though mint is a soulbound that goes to the msg.sender, this could be bypassed with a contract that deploys and calls proxy clones. It gets worse because a limited number of mint also means more affected individuals. More details on the submitted issue.

mint(): If user is issued a irrevocable token, while staking, stakedAt is not set to zero. For more details, refer to the QA report.

##### In PrimeLiquidityProvider.sol:

accrueTokens() guarantees the tokenAccrued variable never surpasses the balance amount, which would lead to a revert due to underflow. This is the only function that increases this variable

setPrimeToken() migrates the prime contract to a new one. This makes it, so the previous contract can't release PLP funds anymore. A certain issue arises from this, for more information, check the QA report.

The contract has a pausable function, if it's paused for too long, accrueTokens() will only return at best 100% of the deposited tokens in the PLP. This is a very edgy case scenario and is very unlikely to happen, though it should be noticed. This could also be prevented by the administrators.

getInterestAccrued() needs a user score, and whenever burn is called for a user, it sets their score to zero. This happens because removed users simply stop getting interest accrued.

It's possible to claimInterest() for other users without their consent. This is a somewhat unorthodox design, but it has no effect on neither the protocol nor the user.

# Codebase quality analysis

The codebase of the project is similar to those that can be found in other BSC smart contracts, this can be seen by checking certain design patterns implemented, for example, internally checking time based parameters through the current block number. 

# Architecture recommendation

The block number architecture must be redesigned when deploying for chains other than opBNB. This is because other chains do not have block times bound by time periods. The protocol should also be aware that the block number amount for opBNB can change in the future.



### Time spent:
25 hours