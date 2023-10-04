## Title
Inconsistent State Transition After Burning Prime Tokens
## Description
In the prime contract, there is an inconsistency in the state transition of a user's reward index after burning prime tokens. Specifically, when a user burns their prime tokens by calling [_burn()](https://github.com/code-423n4/2023-09-venus/blob/9c1016326a0e97376749860266c4541a313a86c2/contracts/Tokens/Prime/Prime.sol#L725-L756), the contract sets the user's `rewardIndex` for all markets to zero. However, when the same user subsequently claims interest by calling [_claimInterest()](https://github.com/code-423n4/2023-09-venus/blob/9c1016326a0e97376749860266c4541a313a86c2/contracts/Tokens/Prime/Prime.sol#L672-L697), the contract resets the rewardIndex to the current market's rewardIndex. This results in an inconsistent state transition.

Here is the sequence of events:

    1) User burns prime tokens by calling `_burn()`.
    2) `_burn()` sets the user's `rewardIndex` for all markets to zero.
    3) Later, the same user calls `_claimInterest()` to claim interest.
    4) `_claimInterest()` resets the user's `rewardIndex` to the current market's `rewardIndex`.

The inconsistency arises because, after burning prime tokens, the user's `rewardIndex` should remain zero until they earn new rewards. However, `_claimInterest()` incorrectly updates it to the current market's `rewardIndex`

maybe this issue could be considered as Medium

## Impact
This will result in invalid accounting .