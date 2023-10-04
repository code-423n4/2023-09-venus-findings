## L-01 Calling _claimInterest() function in Prime.sol re-assigns user's reward index after it was set to 0 when their token was burned
Burning a token sets the user's score and reward index and score to 0 (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L736-L737). Nevertheless after having a token burned the user can still claim the rewards by calling one of the claimInterest functions in Prime.sol (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L433, https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L443).

These functions don't check whether the user still holds a prime token. They also re-assign the user the respective market reward index (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L676) so the users reward index is not more 0.

I have not found a way to exploit this. Potentially this will skew reward calculation for the user once he claims a prime token again. This should be further explored.

## L-02 updateScores() function in Prime.sol reverts when token of user in passed users array is burned prior to calling this function (frontrun)

The function already implements continue instead of revert if the token of a user was already updated in the current update round id (https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L208). This prevents the function to revert if the user was updated in a previous call or is contained more than once in the passed users array.

But the function reverts when the Prime token of a user does not exist (https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L207). This can also happen when a user's token gets burned in the same block (unstakes under minimum staking limit, gets burned manually via protected and public burn function). This could be considered a frontrun.

Under certain circumstance this could even be used to DoS the score updates for a group of users (the passed array of users for which the transaction reverts). A user contained in the users array would need to observe the public mempool and frontrun the updateScores() call to not have an existing prime token anymore.

Since getting a prime token in the first place (at least through self-minting) requires a minimum stake of 1000 XVS (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L34; ~$5000 at current $5/XVS token) and tokens cannot immediately be restaked (needs 90 days minimum staking period) this seems not very likely to be used for DoS but it is certainly possible.

To solve the explained issue the updateScores () function could implement the following instead;
```Solidity
  if (!tokens[user].exists || isScoreUpdated[nextScoreUpdateRoundId][user]) continue;
```

## L-03 Prime.sol issue() issue function would be more versatile using an array parameter for isIrrevocable
Changing the interface of the issue() function (https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L331) to the following would allow issuing irrevocable and revocable tokens with just 1 function call:

```Solidity
function issue(bool[] isIrrevocable, address[] calldata users) external {}
```

It would be necessary though to validate that both arrays have the same length. Using 2 arrays (one for contract/EOA addresses) and another for matching a parameter for each address entry is a common pattern and could be implemented here.

## L-04 Comment for isIrrevocable param of Prime.sol issue() function seems wrong
The tokens are "issued" in either case independent of whether it is an irrevocable token or a revocable token (https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L328). Comment should rather be: "Are issued tokens irrevocable ones" or "Issued tokens are irrevocable".

## L-05 Inconsistent pattern used for resetting stakedAt to 0 in Prime.sol
In Prime.sol the stakedAt of a user is reset to 0 in more than 1 place. But different patterns are used to reset the value to 0. In https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L352 the value is deleted. In https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L376 the value is assign 0. One of both patterns should be used exclusively to reach consistency.

## L-06 Renaming variables and slight restructuring would give accrueInterest(address vToken) function in Prime.sol more clarity
The "distributionIncome" variable (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L568) should better be renamed to sth. like "unreleasedIncome" or "unreleasedPrimeIncome". The following would give the function more clarity (at the expense of 1 additional local variable:
```Solidity
uint256 unreleasedPSRIncome = totalIncomeUnreleased - unreleasedPSRIncome[underlying];
...
uint256 totalAccruedInPLP = _primeLiquidityProvider.tokenAmountAccrued(underlying);
uint256 unreleasedPLPAccruedInterest = totalAccruedInPLP - unreleasedPLPIncome[underlying];

uint256 unreleasedIncome = unreleasedPSRIncome + totalAccruedInPLP;
```

## L-07 local isMarketExist variable inside of addMarket() function of Prime.sol should be renamed
It feels like the naming is grammatically wrong. The variable should be renamed to sth. like "isMarketExisting", "doesMarketExist", "marketExisting", "marketExists" here https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L292.

## L-08 _executeBoost has wrong comment indicating it needs to be called "before changing account's borrow or supply balance."
The comment (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L775) should be "Update total score of user and market. Must be called after changing account's borrow or supply balance." like commented for the _updateScore function (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L790).

## L-09 TokenDistributionSpeedUpdated event in PrimeLiquidityProvider.sol could also log the old distribution speed
The PrimeLiquidityProvider event only logs the new speed (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L33) whereas the PrimeTokenUpdated event also logs the old prime token when logging the new prime token (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L36). Logging the old and the new value for all "*Updated" events could be a pattern worth following throughout the codebase.

## L-10 TokenNotInitialized event in PrimeLiquidityProvider.sol uses parameter with trailing underscore
TokenNotInitialized uses the "token_" parameter with trailing underscore (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L75). Other events use the "token" parameter without underscore (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L66). The underscore should be removed.

## L-11 The tokens_ parameter of initialize() function in PrimeLiquidityProvider.sol could be enhanced with a duplication check
Prior to iterating the passed tokens array or while iterating it (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L103) a duplication check could be introduced making sure a token is not processed multiple times. This would reduced the risk that a token is included more than once but with varying distribution speed which would lead to overwrites.

## L-12 Incrementing the index in loops is not uniform across contracts
For example here "++1" is used (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L108) and here "i++" is used (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L189).

## L-13 The setTokensDistributionSpeed() function should be moved within PrimeLiquidityProvider.sol
It should be placed under the initializeTokens() functions as both functions are required for adding new tokens (same context). I recommend also moving the pauseFundsTransfer() and resumeFundsTransfer() functions (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L132-L144) to the bottom of the contract. They are a dedicated aspect and should not be put between other contract aspects concerned with funds, rewards, etc..

## L-14 It would be beneficial to have a check for not sweeping any of the accrued tokens for sweep() function in PrimeLiquidityProvider.sol
The sweep() function (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216) allows sweeping the full balance of the contract. It only reverts for an attempt to sweep more than the contract balance (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L218). By accident the balance could be corrected to an amount that does not cover the accrued tokens in the PLP anymore. The would otherwise lead to an underflow here (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L261) and a revert trying to send more tokens than available here (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L204).

## L-15 tokenAccrued variable in accrueTokens() functions of PrimeLiquidityProvider.sol could be written a bit cleaner
Currently the following code is used (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L264):
```Solidity
uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);
```
It could be rewritten (semantically) cleaner to use < instead of <=.
```Solidity
uint256 tokenAccrued = (balanceDiff < accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);
```
## L-16 Some NatSpec comments start with a lowercase some with an uppercase letter
Example for lowercase: https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L87
Example for uppercase: https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L60. Should, be consistent throughout the codebase.

## L-17 lessxvsThanCapital variable in Scores.sol does is nor correctly formatted.
The variable should be written "lessXvsThanCapital" instead of "lessxvsThanCapital" (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/Scores.sol#L52)




