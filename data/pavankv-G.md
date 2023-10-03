

## 1. Refactor the whole function to save gas .:-

**i**
Instead of fetching storage variable from other contract storage and copy to storage location in each for loop -iteration will become gas overhead . Dynamic storage array variable `_allMarkets` is used to fetch from contract [`PrimeStorage.sol`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L70) and copy to storage location and [`_allMarkets`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L210) doesn't require any validation, So we can store the [`_allMarkets`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L210) variable out-side the for-loop for one time can be called in entire function .

**ii**
Use do-while loop to simple operation will save 5gas per iteration.

**Before :-**
```solidity
    function updateScores(address[] memory users) external {
        if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
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

            pendingScoreUpdates--;
            isScoreUpdated[nextScoreUpdateRoundId][user] = true;

            unchecked {
                i++;
            }

            emit UserScoreUpdated(user);
        }
    }
```

**After :-**
```solidity
          function updateScores(address[] memory users) external {
        if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
        if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();
        address[] storage _allMarkets = allMarkets;//@audit changed here

        for (uint256 i = 0; i < users.length; ) {
            address user = users[i];

            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

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
    }
```
Please look into `//@audit changed here` comment in above function.

**HardHat Gas Benchmark**

Before
| Contract  | Method   | Min   | Max   | Avg   |
|---|---|---|---|---|
|  Prime |  updateScores | - | - | 646930 |

After
| Contract  | Method   | Min   | Max   | Avg   |
|---|---|---|---|---|
|  Prime |  updateScores | - | - | 505412 |

code snippet:-
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200C5-L226C14

## 2 . In order to save some deployment gas we can check address(0) in if-else only .

The check of address(0) can be done in if-else instead of internal function. Compare to if-else , internal function consumes less gas(4-5) while on calling to check `address(0)` in external or public function. But while deployment of contract internal function(zero-check) consumes more gas(1000 - 1200 and more) than presented if-else statements . In below contract i deployed functions with Internal function check and without internal function check instead of added if-else statements to check with **remix IDE**  below table shows the Gas numbers. :-

**Remix Gas BenchMarks**

| Particulars | With Internal Function  | Without Internal Function |
| :---         |     :---:      |          ---: |
| Gas          | 2,55,027     | 2,51,839    |
| Transaction cost| 2,21,814       | 2,19,045    |
| Execution Cost | 1,57,202 | 1,54,602 |

**HardHat Gas BenchMarks**

Before
| Contract  | Method   | Min   | Max   | Avg   |
|---|---|---|---|---|
|  PrimeLiquidityProvider |  accrueTokens | 44322  | 66230  | 57078  |
| PrimeLiquidityProvider  | initializeTokens  | - | - | 55413  |
| PrimeLiquidityProvider  | setPrimeToken  | 37764  | 54876  |48455 |



After
| Contract  | Method   | Min   | Max   | Avg   |
|---|---|---|---|---|
|  PrimeLiquidityProvider |  accrueTokens | 44293  | 66200  | 55742  |
| PrimeLiquidityProvider  | initializeTokens  | - | - | 55360  |
| PrimeLiquidityProvider  | setPrimeToken  | 37711  | 54823  |48402 |




In below scenario [_ensureZeroAddress](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L344C1-L348C6) function in PrimeLiquidateProvide.sol and called in line [178](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L178),[250](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L250) and [287](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L287) in-order to save deployemnt we can directly use if-else statement .

**Before :-**
```solidity
    function setPrimeToken(address prime_) external onlyOwner {
        _ensureZeroAddress(prime_);//@audit

    function accrueTokens(address token_) public {
        _ensureZeroAddress(token_);//@audit

    function _initializeToken(address token_) internal {
        _ensureZeroAddress(token_);//@audit
 
```
Please see the `//@audit` comment in above function


**After :-**
```solidity
function setPrimeToken(address prime_) external onlyOwner {
       if(prime_ == address(0)) revert("Error");//@audit changed here

function accrueTokens(address token_) public {
       if(token_ == address(0)) revert("Error");//@audit changed here

function _initializeToken(address token_) internal {
         if(token_ == address(0)) revert("Error");//@audit changed here

```
Please look into `//@audit changed here` in above function

code snippet:-
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L287
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L250
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L178
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L344

## 3. Use one time memory location instead of two times :-

In below function we can see `pendingInterests` variable is declared in returns and also in the function . So here there are using two times memory location. We can reduce gas by declaring variable one time and pointing to memory location.

**Before**
```solidity
    function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {//@audit
        address[] storage _allMarkets = allMarkets;
        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);//@audit

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

        return pendingInterests;//@audit
    }
```
Look into `//@audit` comment in above function


**After**

```solidity
    function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {//@audit changed here
        address[] storage _allMarkets = allMarkets;
         pendingInterests = new PendingInterest[](_allMarkets.length);//@audit changed here

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

        //return pendingInterests;//@audit changed here
    }
```

Please look into `//@audit changed here` in above function.

code snippet:-
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174C4-L194C6


## 4. Function with `onlyOwner()` modifier can be marked as payabale :-

It will save gas 21 per call because by marking payable it will prevent the extra OP code `(CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2))`

**Before**
```solidity
function initializeTokens(address[] calldata tokens_) external onlyOwner { //@audit 

function setPrimeToken(address prime_) external onlyOwner {//@audit

function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner { //@audit
```
please look into `//@audit` in above function.

**After**
```solidity
function initializeTokens(address[] calldata tokens_) external payable onlyOwner { //@audit changed here

function setPrimeToken(address prime_) external payable onlyOwner {//@audit changed here

function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external payable onlyOwner { //@audit changed here

```
Please look into `//@audit changed here` in above function

**Hardhat Gas BenchMark**

Before
| Contract  | Method   | Min   | Max   | Avg   |
|---|---|---|---|---|
|  PrimeLiquidityProvider |  initializeTokens | - | -  | 55413  |
| PrimeLiquidityProvider  | setPrimeToken  | 37711 | 54876 | 48455 |
| PrimeLiquidityProvider  | sweepToken  | - | - |46520 |


After
| Contract  | Method   | Min   | Max   | Avg   |
|---|---|---|---|---|
|  PrimeLiquidityProvider |  initializeTokens | - | -  | 55360  |
| PrimeLiquidityProvider  | setPrimeToken  | 37711 | 54823 | 48402 |
| PrimeLiquidityProvider  | sweepToken  | - | - |46496 |

code snippet:-
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118

## 5. Named the returns of internal function to save some deployment gas :-

**Before**
```solidity
 function _claimInterest(address vToken, address user) internal returns (uint256) {//@audit
        uint256 amount = getInterestAccrued(vToken, user);//@audit
        amount += interests[vToken][user].accrued;

        interests[vToken][user].rewardIndex = markets[vToken].rewardIndex;
        interests[vToken][user].accrued = 0;

        address underlying = _getUnderlying(vToken);
        IERC20Upgradeable asset = IERC20Upgradeable(underlying);

        if (amount > asset.balanceOf(address(this))) {
            address[] memory assets = new address[](1);
            assets[0] = address(asset);
            IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
            if (amount > asset.balanceOf(address(this))) {
                IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
                unreleasedPLPIncome[underlying] = 0;
            }
        }

        asset.safeTransfer(user, amount);

        emit InterestClaimed(user, vToken, amount);

        return amount;//@audit
    }
```
Look into `//@audit` comment in above function

**After :-**
```solidity
    function _claimInterest(address vToken, address user) internal returns (uint256 amount) {//@audit changed here
         amount = getInterestAccrued(vToken, user);//@audit changed here
        amount += interests[vToken][user].accrued;

        interests[vToken][user].rewardIndex = markets[vToken].rewardIndex;
        interests[vToken][user].accrued = 0;

        address underlying = _getUnderlying(vToken);
        IERC20Upgradeable asset = IERC20Upgradeable(underlying);

        if (amount > asset.balanceOf(address(this))) {
            address[] memory assets = new address[](1);
            assets[0] = address(asset);
            IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
            if (amount > asset.balanceOf(address(this))) {
                IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
                unreleasedPLPIncome[underlying] = 0;
            }
        }

        asset.safeTransfer(user, amount);

        emit InterestClaimed(user, vToken, amount);

        //return amount;//@audit changed here
    }
```
Please look into `//@audit changed here` comment in above function

code snippet:-
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L672C5-L697C6


## 6 .Use do-while loop for simple operation .

Each iteration will 5 gas.

**Before :-**
```solidity
        for (uint256 i = 0; i < allMarkets.length; ) {//@audit
            accrueInterest(allMarkets[i]);

            unchecked {
                i++;
            }
        }

```
Look into `//@audit` comment in above function

**After :-**
```solidity
  uint i; //@audit changed here
  do{
       accrueInterest(allMarkets[i]);
       ++i;
    }while(i < allMarkets.length);//@audit changed here

```

Look `//@audit changed here` comment in above comment.

code snippet:-
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246C1-L252C10

scenario 2 :-
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L609C1-L616C10


