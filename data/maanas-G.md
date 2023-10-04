### Summary


|ID|Title|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [GAS-01](#gas-01-use-pre-incrementdecrement-i--i-instead-of-post-incrementdecrement-ii--)| Use pre-increment/decrement (++i/--i) instead of post-increment/decrement (i++/i--) | 14 | 70 |
| [GAS-02](#gas-02-use-oz-unsafeaccess-to-avoid-overhead-associated-with-access-of-storage-array-index)| Use OZ `unsafeAccess` to avoid overhead associated with access of storage array index | 2 | 244 |
| [GAS-03](#gas-03-wherever-the-operation-is-not-going-to-overflowunderflow-use-unchecked)| Wherever the operation is not going to overflow/underflow, use unchecked | 1 | 130 |
| [GAS-04](#gas-04-cache-state-variables-rather-than-re-reading-them-from-storage-in-loop)| Cache state variables rather than re-reading them from storage in loop | 1 | 1402 |
| [GAS-05](#gas-05-cache-array-items-used-multiple-times)| Cache array items used multiple times | 3 | 644 |
| [GAS-06](#gas-06-unwanted-storing-of-data)| Unwanted storing of data | 1 | 10 |
| [GAS-07](#gas-07-assigning-to-default-value)| Assigning to default value | 1 | - |


Total: 23 instances over 7 issues with 4627 gas saved


## Gas Optimizations

### [GAS-01] Use pre-increment/decrement (++i/--i) instead of post-increment/decrement (i++/i--)
*Saves 5 gas*

*There are 14 instances of this issue*

<details>
<summary>see instances</summary>

```solidity
File: contracts/Tokens/Prime/Prime.sol

189:                        i++; 


217:                            j++; 


221:                    pendingScoreUpdates--;


225:                        i++; 


250:                        i++; 


345:                            i++; 


355:                            i++; 


636:                        i++; 


711:                    totalIrrevocable++;


713:                    totalRevocable++; 


740:                        i++; 


745:                    totalIrrevocable--; 


747:                    totalRevocable--; 


828:                if (totalScoreUpdatesRequired > 0) totalScoreUpdatesRequired--; // @audit mns-pre-increment

```

*GitHub*: [189](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L189-#L189), [217](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L217-#L217), [221](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L221-#L221), [225](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L225-#L225), [250](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L250-#L250), [345](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L345-#L345), [355](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L355-#L355), [636](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L636-#L636), [711](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L711-#L711), [713](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L713-#L713), [740](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L740-#L740), [745](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L745-#L745), [747](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L747-#L747), [828](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L828-#L828)


</details>

### [GAS-02] Use OZ `unsafeAccess` to avoid overhead associated with access of storage array index
Since out-of-bound indexes are managed, gas can be saved by using OpenZeppelin's `unsafeAccess` method from [Arrays.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Arrays.sol).

*There are 2 instances of this issue*

```solidity
File: contracts/Tokens/Prime/Prime.sol

609:                for (uint256 i = 0; i < _allMarkets.length; ) {  
610:                    _executeBoost(user, _allMarkets[i]);


730:                for (uint256 i = 0; i < _allMarkets.length; ) { 
731:                    _executeBoost(user, _allMarkets[i]); 

```

*GitHub*: [609](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L609-#L610), [730](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L730-#L731)


### [GAS-03] Wherever the operation is not going to overflow/underflow, use unchecked
Unchecked eliminates the overflow/underflow checks that would be perfromed on normal operations

*There are 1 instances of this issue*

```solidity
File: contracts/Tokens/Prime/Prime.sol

200:            function updateScores(address[] memory users) external {
201:                if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
202:                if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();             
                /// more code 
221:                    pendingScoreUpdates--; // @audit pendingScoreUpdates == 0 is present in the top of the function

```

*GitHub*: [200](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-#L221)


### [GAS-04] Cache state variables rather than re-reading them from storage in loop
Currently `_allMarkets` is read from storage for every user. Caching to outside the `users` for loop will hence save storage reads for every subsequent user.

*There are 1 instances of this issue*

```solidity
File: contracts/Tokens/Prime/Prime.sol

204:                for (uint256 i = 0; i < users.length; ) {
205:                    address user = users[i];
206:        
207:                    if (!tokens[user].exists) revert UserHasNoPrimeToken();
208:                    if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue; 
209:        
210:                    address[] storage _allMarkets = allMarkets; // @audit cache-variable
211:                    for (uint256 j = 0; j < _allMarkets.length; ) {

```

*GitHub*: [204](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L204-#L211)


### [GAS-05] Cache array items used multiple times
Since these items are used multiple times, they can be cache to reduce the overhead with array access.
*There are 3 instances of this issue*

```solidity
File: contracts/Tokens/Prime/Prime.sol

335:                    for (uint256 i = 0; i < users.length; ) { // @audit  cache-variable
336:                        Token storage userToken = tokens[users[i]];
337:                        if (userToken.exists && !userToken.isIrrevocable) {
338:                            _upgrade(users[i]);
339:                        } else {
340:                            _mint(true, users[i]);
341:                            _initializeMarkets(users[i]);
342:                        }


349:                    for (uint256 i = 0; i < users.length; ) { // @audit  cache-variable
350:                        _mint(false, users[i]);
351:                        _initializeMarkets(users[i]);
352:                        delete stakedAt[users[i]];

```

*GitHub*: [335](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L335-#L342), [349](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L349-#L352)
```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

161:                for (uint256 i; i < numTokens; ) { // @audit cache-variable
162:                    _ensureTokenInitialized(tokens_[i]);
163:                    _setTokenDistributionSpeed(tokens_[i], distributionSpeeds_[i]);

```

*GitHub*: [161](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L161-#L163)


### [GAS-06] Unwanted storing of data
Since the return value is used only once, it can be directly used instead of saving it into a variable

*There are 1 instances of this issue*

```solidity
File: contracts/Tokens/Prime/Prime.sol

292:                bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken); // @audit do-direct-check
293:                if (!isMarketExist) revert InvalidVToken();
294:        

```

*GitHub*: [292](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L292-#L294)


### [GAS-07] Assigning to default value
Saves 1284 deployment gas
*There are 1 instances of this issue*

```solidity
File: contracts/Tokens/Prime/Prime.sol

156:                nextScoreUpdateRoundId = 0;

```

*GitHub*: [156](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L156-#L156)