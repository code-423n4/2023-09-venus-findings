# [1] StakeAt missing to be set to 0

## Link 
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L340

## Impact
If a user stakes and his `stakedAt` starts working. If for some reason the protocol wants to issue an irrevocable token to this user with the "issue()" function, `stakedAt` is not set to 0 .

If the protocol decides to `burn()` the token I assign to this user, the user can `claim()` later if they meet the requirements, because skedAt still works and they don't have to wait 90 days.


 