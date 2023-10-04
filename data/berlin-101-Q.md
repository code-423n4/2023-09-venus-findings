## L-01 updateScores() function in Prime.sol could revert when token of user in passed users array is burned prior to calling this function

The function already implements continue instead of revert if the token of a user was already updated in the current update round id (https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L208). This prevents the function to revert if the user was updated in a previous call or is contained more than once in the passed users array.

But the function reverts when the Prime token of a user does not exist (https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L207). This can also happen when a user's token gets burned in the same block (unstakes under minimum staking limit, gets burned manually via protected and public burn function). This could be considered a frontrun.

Under certain circumstance this could even be used to DoS the score updates for a group of users (the passed array of users for which the transaction reverts). A user contained in the users array would need to observe the public mempool and frontrun the updateScores() call to not have an existing prime token anymore.

Since getting a prime token in the first place (at least through self-minting) requires a minimum stake of 1000 XVS (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L34; ~$5000 at current $5/XVS token) and tokens cannot immediately be restaked (needs 90 days minimum staking period) this seems not very likely to be used for DoS but it is certainly possible.

To solve the explained issue the updateScores () function could implement the following instead;
```Solidity
  if (!tokens[user].exists || isScoreUpdated[nextScoreUpdateRoundId][user]) continue;
```

## L-02 Prime.sol issue() issue function would be more versatile using an array parameter for isIrrevocable
Changing the interface of the issue() function (https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L331) to the following would allow issuing irrevocable and revocable tokens with just 1 function call:

```Solidity
function issue(bool[] isIrrevocable, address[] calldata users) external {}
```

It would be necessary though to validate that both arrays have the same length. Using 2 arrays (one for contract/EOA addresses) and another for matching a parameter for each address entry is a common pattern and could be implemented here.

## L-03 Comment for isIrrevocable param of Prime.sol issue() function seems wrong
The tokens are "issued" in either case independent of whether it is an irrevocable token or a revocable token (https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L328). Comment should rather be: "Are issued tokens irrevocable ones" or "Issued tokens are irrevocable".

## L-04
