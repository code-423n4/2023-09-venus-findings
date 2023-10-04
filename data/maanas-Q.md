## Summary

### Low Risk Issues


|ID|Title|Instances|
|-|:-|:-:|
| [L-01](#l-01-continue-before-loop-variable-increment)| `continue` before loop variable increment | 1 |


Total: 1 instances over 1 issues

### Non-critical Issues


|ID|Title|Instances|
|-|:-|:-:|
| [NC-01](#nc-01-variable-redeclaration)| Variable Redeclaration | 1 |
| [NC-02](#nc-02-prefer-named-return-values)| Prefer named return values | 1 |


Total: 2 instances over 2 issues

## Low Risk Issues

### [L-01] `continue` before loop variable increment
In `updateScores` function, the rest of the loop execution is skipped with `continue` if a user's score has already been updated. But the updation of the loop variable occurs after this line resulting in an infinite loop until the tx runs out of gas.
```solidity
        for (uint256 i = 0; i < users.length; ) {
            address user = users[i];

            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

           ///// more code
            
            unchecked {
                i++;
            }
```

*GitHub*: [208](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L208-#L208)


## Non-critical Issues

### [NC-01] Variable Redeclaration
Since `pendingInterests` has been declared as the return variable in the function definition, it is not required to redeclare it within the function. 

*There are 1 instances of this issue*

```solidity
File: contracts/Tokens/Prime/Prime.sol

174:            function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
175:                address[] storage _allMarkets = allMarkets;
176:                PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length); 

```

*GitHub*: [174](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-#L176)


### [NC-02] Prefer named return values
Using named return values can provide more clarity to the function definition

*There are 1 instances of this issue*

```solidity
File: contracts/Tokens/Prime/Prime.sol

872:            function _capitalForScore(
873:                uint256 xvs,
874:                uint256 borrow,
875:                uint256 supply,
876:                address market
877:            ) internal view returns (uint256, uint256, uint256) { 

```

*GitHub*: [872](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L872-#L877)