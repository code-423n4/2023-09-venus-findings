
### Gas Optimizations List

| Number | Optimization Details                                                                         | Instances |
| :----: | :------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Cache state variables outside of loop to avoid reading/writing storage on every iteration.                                                          |     1     |
| [G-02] | No need to create Function to get block.number and No need to cache it                                                       |     1     |
| [G-03] | State variables can be cached instead of re-reading them from storage.                                         |     2     |
| [G-04] | Functions guaranteed to revert when called by normal users can be marked payable                                  |   3    |
| [G-05] | Do not calculate constants to save gas.                                          |     3     |
| [G-06] | Don’t cache value if it is only used once                                                    |     2     |
| [G-07] | No need to emit state variable                           |     1     |
| [G-08] | Write for loops in more gas efficient way |    9     |
| [G-09] | Avoid copying same storage pointer pointing same array                               |     5     |
| [G-10] | Use += / -= for mappings             |    4     |
| [G-11] | Using `ternary` operator instead of single line if-else saves gas             |    7     |
| [G-12] | OR in `if-`condition can be rewritten to two single `if` conditions             |    5     |

Total 12 issues


## [G-01] Cache state variables outside of loop to avoid reading/writing storage on every iteration.

Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a Gwarmaccess (100 gas) per loop iteration. In addition, update the storage variable outside the loop to save 1 SSTORE per loop iteration.

*_There is 1 Instance of this issue_*

### Cache `pendingScoreUpdates` outside of the loop to save 1 SSTORE and 1 SLOAD per iteration, update it's state variable after the loop ended.

*Note: Bot report saves only SLOAD but here we saves 1 SSTORE along  with 1 SLOAD on each iteration*

```solidity
File : contracts/Tokens/Prime/Prime.sol

200: function updateScores(address[] memory users) external {
201:        if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
202:        if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();
203:
204:        for (uint256 i = 0; i < users.length; ) {
205:            address user = users[i];
206:
207:            if (!tokens[user].exists) revert UserHasNoPrimeToken();
208:            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;
209:
210:            address[] storage _allMarkets = allMarkets;
211:            for (uint256 j = 0; j < _allMarkets.length; ) {
212:                address market = _allMarkets[j];
213:                _executeBoost(user, market);
214:                _updateScore(user, market);
215:
216:                unchecked {
217:                    j++;
218:                }
219:            }
220:
221:            pendingScoreUpdates--;
222:            isScoreUpdated[nextScoreUpdateRoundId][user] = true;
223:
224:            unchecked {
225:                i++;
226:            }
227:
228:            emit UserScoreUpdated(user);
229:        }
230:    }

```
[200-230](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200C5-L230C6)

```diff
File : contracts/Tokens/Prime/Prime.sol

function updateScores(address[] memory users) external {

+       uint256 _pendingScoreUpdates = pendingScoreUpdates; 

-       if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
+       if (_pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
        if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

        for (uint256 i = 0; i < users.length; ) {
            address user = users[i];

            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

            address[] storage _allMarkets = allMarkets;
            for (uint256 j = 0; j < _allMarkets.length; ) {
                address market = _allMarkets[j];
                _executeBoost(user, market);
                _updateScore(user, market);

                unchecked {
                    j++;
                }
            }

-           pendingScoreUpdates--;
+           _pendingScoreUpdates--;
            isScoreUpdated[nextScoreUpdateRoundId][user] = true;

            unchecked {
                i++;
            }

            emit UserScoreUpdated(user);
        }
+       pendingScoreUpdates = _pendingScoreUpdates;        
    }

```

## [G-02] No need to create Function to get block.number and No need to cache it

'No need to create Function to get block.number and No need to cache it' this statement means that we don't need to create a function to get block.number because block.number is a global variable so we also don't need to cache it . We can use directly block.number instead of caching and creating function for it .

```solidity
File : contracts/Tokens/Prime/PrimeLiquidityProvider.sol

249: function accrueTokens(address token_) public {
        _ensureZeroAddress(token_);

        _ensureTokenInitialized(token_);

        uint256 blockNumber = getBlockNumber();
        uint256 deltaBlocks = blockNumber - lastAccruedBlock[token_];

        if (deltaBlocks > 0) {
            uint256 distributionSpeed = tokenDistributionSpeeds[token_];
            uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

            uint256 balanceDiff = balance - tokenAmountAccrued[token_];
            if (distributionSpeed > 0 && balanceDiff > 0) {
                uint256 accruedSinceUpdate = deltaBlocks * distributionSpeed;
                uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);

                tokenAmountAccrued[token_] += tokenAccrued;
                emit TokensAccrued(token_, tokenAccrued);
            }

            lastAccruedBlock[token_] = blockNumber;
         }
     }

     /// @notice Get the latest block number
     /// @return blockNumber returns the block number
     function getBlockNumber() public view virtual returns (uint256) {
        return block.number;
     }

     /**
     * @notice Initialize the distribution of the token
     * @param token_ Address of the token to be intialized
     * @custom:event Emits TokenDistributionInitialized event
     * @custom:error Throw TokenAlreadyInitialized if token is already initialized
     */
     function _initializeToken(address token_) internal {
        _ensureZeroAddress(token_);
        uint256 blockNumber = getBlockNumber();
        uint256 initializedBlock = lastAccruedBlock[token_];

        if (initializedBlock > 0) {
            revert TokenAlreadyInitialized(token_);
        }

        /*
         * Update token state block number
         */
        lastAccruedBlock[token_] = blockNumber;

        emit TokenDistributionInitialized(token_);
301:    }

```
[249-301](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249C5-L301C6)

```diff
File : contracts/Tokens/Prime/PrimeLiquidityProvider.sol

function accrueTokens(address token_) public {
        _ensureZeroAddress(token_);

        _ensureTokenInitialized(token_);

-       uint256 blockNumber = getBlockNumber();
-       uint256 deltaBlocks = blockNumber - lastAccruedBlock[token_];
+       uint256 deltaBlocks = block.number - lastAccruedBlock[token_];

        if (deltaBlocks > 0) {
            uint256 distributionSpeed = tokenDistributionSpeeds[token_];
            uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

            uint256 balanceDiff = balance - tokenAmountAccrued[token_];
            if (distributionSpeed > 0 && balanceDiff > 0) {
                uint256 accruedSinceUpdate = deltaBlocks * distributionSpeed;
                uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);

                tokenAmountAccrued[token_] += tokenAccrued;
                emit TokensAccrued(token_, tokenAccrued);
            }

-           lastAccruedBlock[token_] = blockNumber;
+           lastAccruedBlock[token_] = block.number;
        }
    }

-    /// @notice Get the latest block number
-    /// @return blockNumber returns the block number
-   function getBlockNumber() public view virtual returns (uint256) {
-       return block.number;
-    }

    /**
     * @notice Initialize the distribution of the token
     * @param token_ Address of the token to be intialized
     * @custom:event Emits TokenDistributionInitialized event
     * @custom:error Throw TokenAlreadyInitialized if token is already initialized
     */
    function _initializeToken(address token_) internal {
        _ensureZeroAddress(token_);
-       uint256 blockNumber = getBlockNumber();
        uint256 initializedBlock = lastAccruedBlock[token_];

        if (initializedBlock > 0) {
            revert TokenAlreadyInitialized(token_);
        }

        /*
         * Update token state block number
         */
-       lastAccruedBlock[token_] = blockNumber;
+       lastAccruedBlock[token_] = block.number;

        emit TokenDistributionInitialized(token_);
    }

```


## [G-03] State variables can be cached instead of re-reading them from storage.

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

*`Note: These instances missed by bot report`*

*_There are 2 Instances of this issue_*

### Cache `totalIrrevocable + totalRevocable` to save 2 SLOAD and 1 ADD opcode
```solidity
File : contracts/Tokens/Prime/Prime.sol

818:  function _startScoreUpdateRound() internal {
819:      nextScoreUpdateRoundId++;
820:      totalScoreUpdatesRequired = totalIrrevocable + totalRevocable;
821:      pendingScoreUpdates = totalScoreUpdatesRequired;
822:  }

```
[818-822](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L818C1-L822C6)

```diff
File : contracts/Tokens/Prime/Prime.sol

    function _startScoreUpdateRound() internal {
        nextScoreUpdateRoundId++;
+       uint256 _totalScoreUpdatesRequired = totalIrrevocable + totalRevocable;  
-       totalScoreUpdatesRequired = totalIrrevocable + totalRevocable;
+       totalScoreUpdatesRequired = _totalScoreUpdatesRequired;
-       pendingScoreUpdates = totalScoreUpdatesRequired;
+       pendingScoreUpdates = _totalScoreUpdatesRequired;
    }

```

## [G‑04] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

*_There are 3 Instances of this issue_*
```solidity
File : contracts/Tokens/Prime/PrimeLiquidityProvider.sol

118: function initializeTokens(address[] calldata tokens_) external onlyOwner {

177: function setPrimeToken(address prime_) external onlyOwner {

216: function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {

```
[118](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118), [177](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177), [216](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216)

```diff
File : contracts/Tokens/Prime/PrimeLiquidityProvider.sol

- function initializeTokens(address[] calldata tokens_) external onlyOwner {
+ function initializeTokens(address[] calldata tokens_) external payable onlyOwner {

- function setPrimeToken(address prime_) external onlyOwner {
+ function setPrimeToken(address prime_) external payable onlyOwner {

- function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
+ function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external payable onlyOwner {

```

## [G-05] Do not calculate constants to save gas.

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

In Solidity, a constant is a value that is known at compile-time and cannot be changed during runtime. Constants can be defined using the constant keyword or the immutable keyword. The statement "Do not calculate constants to save gas" means that if a constant value can be calculated at compile-time, it is more gas-efficient to define the constant directly with the calculated value rather than calculating it at runtime. This is because calculating a constant at runtime requires additional gas to be consumed, whereas defining it directly with the calculated value does not.

*_There are 3 Instances of this issue_*
```solidity
File : contracts/Tokens/Prime/PrimeStorage.sol

34: uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;

37: uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

40: uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;

```
[34-40](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L34C1-L40C64)



## [G-06] Don’t cache value if it is only used once

If a value is only intended to be used once then it should not be cached. Caching the value will result in unnecessary stack manipulation.

*_There are 2 Instances of this issue_*
```solidity
File : contracts/Tokens/Prime/PrimeLiquidityProvider.sol

289:  uint256 initializedBlock = lastAccruedBlock[token_];
290:
291:  if (initializedBlock > 0) {
292:     revert TokenAlreadyInitialized(token_);
293:     }

```
[289-293](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L289C1-L293C10)

```solidity
File : contracts/Tokens/Prime/PrimeLiquidityProvider.sol

333:  uint256 lastBlockAccrued = lastAccruedBlock[token_];
334:
335:   if (lastBlockAccrued == 0) {
336:     revert TokenNotInitialized(token_);
337:   }

```
[333-337](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L333C8-L337C10)


## [G-07] No need to emit state variable

### Emit `_comptroller` instead of `comptroller` because of `if (comptroller != _comptroller) revert InvalidComptroller();` this condition check.

*_There is 1 Instance of this issue_*
```solidity
File : contracts/Tokens/Prime/Prime.sol

454:    if (comptroller != _comptroller) revert InvalidComptroller();
455:
456:    address vToken = vTokenForAsset[asset];
457:    if (vToken == address(0)) revert MarketNotSupported();
458:
459:    IVToken market = IVToken(vToken);
460:    unreleasedPSRIncome[_getUnderlying(address(market))] = 0;
461:
462:    emit UpdatedAssetsState(comptroller, asset);

```
[454-462](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L454C9-L462C53)

```diff
File : contracts/Tokens/Prime/Prime.sol

function updateAssetsState(address _comptroller, address asset) external {
        if (msg.sender != protocolShareReserve) revert InvalidCaller();
        if (comptroller != _comptroller) revert InvalidComptroller();

        address vToken = vTokenForAsset[asset];
        if (vToken == address(0)) revert MarketNotSupported();

        IVToken market = IVToken(vToken);
        unreleasedPSRIncome[_getUnderlying(address(market))] = 0;

-       emit UpdatedAssetsState(comptroller, asset);
+       emit UpdatedAssetsState(_comptroller, asset);
    }

```

## [G-08] Write for loops in more gas efficient way

Loop can be written in more gas efficient way, by

1. taking length in stack variable,  //@audit this is included in bot 
2. using unchecked{++i} with pre increment
3. not re-initializing to 0 since it is default
4. Using <= is cheaper than <     //@audit this is included in bot 

*_There are 9 Instances of this issue_*

This is what a gas-optimal for loop looks like, if you combine the two tricks above:
```solidity

for (uint256 i; i < limit; ) {
    
    // inside the loop
    
    unchecked {
        ++i;
    }
}

```


```solidity
File : contracts/Tokens/Prime/Prime.sol

for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            uint256 interestAccrued = getInterestAccrued(market, user);
            uint256 accrued = interests[market][user].accrued;

            pendingInterests[i] = PendingInterest({
                market: IVToken(market).underlying(),
                amount: interestAccrued + accrued
            });

            unchecked {
                i++;
            }
        }

```
[178-190](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L178C9-L190C14)

```solidity
File : contracts/Tokens/Prime/Prime.sol

for (uint256 i = 0; i < users.length; ) {
            address user = users[i];

            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

            address[] storage _allMarkets = allMarkets;
            for (uint256 j = 0; j < _allMarkets.length; ) {
                address market = _allMarkets[j];
                _executeBoost(user, market);
                _updateScore(user, market);

                unchecked {
                    j++;
                }
            }

            pendingScoreUpdates--;
            isScoreUpdated[nextScoreUpdateRoundId][user] = true;

            unchecked {
                i++;
            }

            emit UserScoreUpdated(user);
        }

```
[204-229](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L204C9-L229C10)

```solidity
File : contracts/Tokens/Prime/Prime.sol

for (uint256 i = 0; i < allMarkets.length; ) {
            accrueInterest(allMarkets[i]);

            unchecked {
                i++;
            }
        }

```
[246-252](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246C9-L252C10)

```solidity
File : contracts/Tokens/Prime/Prime.sol

if (isIrrevocable) {
            for (uint256 i = 0; i < users.length; ) {
                Token storage userToken = tokens[users[i]];
                if (userToken.exists && !userToken.isIrrevocable) {
                    _upgrade(users[i]);
                } else {
                    _mint(true, users[i]);
                    _initializeMarkets(users[i]);
                }

                unchecked {
                    i++;
                }
            }
        } else {
            for (uint256 i = 0; i < users.length; ) {
                _mint(false, users[i]);
                _initializeMarkets(users[i]);
                delete stakedAt[users[i]];

                unchecked {
                    i++;
                }
            }
        }

```
[334-358](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L334C9-L358C10)

```solidity
File : contracts/Tokens/Prime/Prime.sol

for (uint256 i = 0; i < _allMarkets.length; ) {
            _executeBoost(user, _allMarkets[i]);
            _updateScore(user, _allMarkets[i]);

            unchecked {
                i++;
            }
        }

```
[609-616](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L609C9-L616C10)

```solidity
File : contracts/Tokens/Prime/Prime.sol

for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            accrueInterest(market);

            interests[market][account].rewardIndex = markets[market].rewardIndex;

            uint256 score = _calculateScore(market, account);
            interests[market][account].score = score;
            markets[market].sumOfMembersScore = markets[market].sumOfMembersScore + score;

            unchecked {
                i++;
            }
        }

```
[625-638](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L625C9-L638C10)

```solidity
File : contracts/Tokens/Prime/Prime.sol

for (uint256 i = 0; i < _allMarkets.length; ) {
            _executeBoost(user, _allMarkets[i]);

            markets[_allMarkets[i]].sumOfMembersScore =
                markets[_allMarkets[i]].sumOfMembersScore -
                interests[_allMarkets[i]][user].score;
            interests[_allMarkets[i]][user].score = 0;
            interests[_allMarkets[i]][user].rewardIndex = 0;

            unchecked {
                i++;
            }
        }

```
[730-742](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L730C9-L742C10)

## [G-09] Avoid copying same storage pointer pointing same array 

Unnecessary making `_allMarkets` it can be accessed simply from existing `allMarkets`. Here `_allMarkets` and `allMarkets` both pointing to same storage array so no difference between them.

*_There are 5 Instances of this issue_*
```solidity
File : contracts/Tokens/Prime/Prime.sol

175: address[] storage _allMarkets = allMarkets;

210: address[] storage _allMarkets = allMarkets;

608: address[] storage _allMarkets = allMarkets;

624: address[] storage _allMarkets = allMarkets;

728: address[] storage _allMarkets = allMarkets;

```
[175](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L175), [210](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L210), [608](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L608), [624](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L624), [728](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L728)

## [G‑10] Use += / -= for mappings

Using += / -= for mappings saves 40 gas due to not having to recalculate the mapping's value's hash.

*_There are 4 Instances of this issue_*
```solidity
File : contracts/Tokens/Prime/Prime.sol

588: markets[vToken].rewardIndex = markets[vToken].rewardIndex + delta;

633: markets[market].sumOfMembersScore = markets[market].sumOfMembersScore + score;

733: markets[_allMarkets[i]].sumOfMembersScore =
734:    markets[_allMarkets[i]].sumOfMembersScore -
735:    interests[_allMarkets[i]][user].score;

800: markets[market].sumOfMembersScore = markets[market].sumOfMembersScore - interests[market][user].score + score;

```
[588](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L588), [633](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L633), [733-735](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L733C1-L735C55), [800](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L800)


## [G-11] Using `ternary` operator instead of single line if-else saves gas

When using the if-else construct, Solidity generates bytecode that includes a JUMP operation. The JUMP operation requires the EVM to change the program counter to a different location in the bytecode, resulting in additional gas costs. On the other hand, the ternary operator doesn't involve a JUMP operation, as it generates bytecode that utilizes conditional stack manipulation.
The exact amount of gas saved by using the ternary operator instead of the if-else construct in Solidity can vary depending on the specific scenario, the complexity of the surrounding code, and the conditions involved.

*_There are 7 Instances of this issue_*
```solidity
File : contracts/Tokens/Prime/Prime.sol

370: if (tokens[user].isIrrevocable) {
371:    _accrueInterestAndUpdateScore(user);
372:   } else {
373:      _burn(user);
374:     }

421: if (paused()) {
422:        _unpause();
423:    } else {
424:      _pause();
425:     }

482: if (totalTimeStaked < STAKING_PERIOD) {
483:       return STAKING_PERIOD - totalTimeStaked;
484:    } else {
485:       return 0;
486:    }

710: if (isIrrevocable) {
711:       totalIrrevocable++;
712:   } else {
713:       totalRevocable++;
714:      }

744:  if (tokens[user].isIrrevocable) {
745:      totalIrrevocable--;
746:   } else {
747:      totalRevocable--;
748:    }

855: if (xvs > MAXIMUM_XVS_CAP) {
856:        return MAXIMUM_XVS_CAP;
857:    } else {
858:       return xvs;
859:    }

931: if (vToken == VBNB) {
932:       return WBNB;
933:    } else {
934:       return IVToken(vToken).underlying();
935:    }

```
[370-374](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L370C13-L374C14), [421-425](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L421C9-L425C10), [482-486](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L482C9-L486C10), [710-714](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L710C9-L714C10), [744-748](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L744C8-L748C10), [855-859](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L855C9-L859C10), [931-935](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L931C9-L935C10) 


## [G-12] OR in `if-`condition can be rewritten to two single `if` conditions

 Refactoring the if-condition in a way it won’t be containing the || operator will save more gas.

For Example:
```solidity
contract CustomErrorBoolLessEfficient {
    error BadValue();

    function requireGood(uint256 x) external pure {
        if (x < 10 || x > 20) {
            revert BadValue();
        }
    }
}

contract CustomErrorBoolEfficient {
    error TooLow();
    error TooHigh();

    function requireGood(uint256 x) external pure {
        if (x < 10) {
            revert TooLow();
        }
        if (x > 20) {
            revert TooHigh();
        }
    }
}
```

```solidity
File : contracts/Tokens/Prime/Prime.sol

318: if (_irrevocableLimit < totalIrrevocable || _revocableLimit < totalRevocable) revert InvalidLimit();

716: if (totalIrrevocable > irrevocableLimit || totalRevocable > revocableLimit) revert InvalidLimit();

780: if (!markets[vToken].exists || !tokens[user].exists) {
781:      return;
782:     }

795: if (!markets[market].exists || !tokens[user].exists) {
796:     return;
797:   }

810: if (_alphaDenominator == 0 || _alphaNumerator > _alphaDenominator) {
811:         revert InvalidAlphaArguments();
812:       }

```
[318](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L318), [716](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L716), [780-782](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L780C9-L782C10), [810-812](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L810C9-L812C10)
