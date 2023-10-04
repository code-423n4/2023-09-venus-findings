# Codebase Optimization Report



## Table Of Contents

  - [\[G-01\] Using immutable on variables that are only set in the constructor and never after ](#g-)
  - [\[G-02\] Variable should created outside of loops](#g-)
  - [\[G-03\] `address[] storage _allMarkets` could made as `calldata` instead `storage` in multiple instances](#g-)
  - [\[G-04\] Some State Variable could cached in `memory` with out calling them repeatedly (Those are missed by automated finding reports)](#g-)
  - [\[G-05\] If()/require() checks which make state variable or important variable checks should be at top of function.](#g-)
  - [\[G-06\] To update one limit (irrevocableLimit or revocableLimit) other automatically get override](#g-)
  - [\[G-07\] Use `delete` key word instead of setting mapping variables to default](#g-)
  - [\[G-08\] Use assembly for some more check work can save some extra gas](#g-)
      - [\[G-08-01\] Use to check `msg.sender`](#g-)
      - [\[G-08-02\] Use to check against some `state variables`](#g-)
  - [\[G-09\] Ternary Operator could used when ever possible](#g-)
  - [\[G-10\] Some Mapping value could be cached into `storage`](#g-)
  - [\[G-11\] A extra storage call will be save by caching mathematical operation in memory](#g-)
  - [\[G-12\] Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement (These are missing in automated finding reports)](#g-)
      - [\[G-12-01\] `totalScoreUpdatesRequired--` could be uncheked because of previous `if (totalScoreUpdatesRequired > 0)` check](#g-)
      - [\[G-12-02\] Due to `pendingScoreUpdates > 0` check `pendingScoreUpdates--` could be unchecked](#g-)
  - [\[G-13\] `address(this)` could cached in a `immutable` state variable](#g-)
  - [\[G-14\] `struct` can be packed more precisely](#g-)
      - [\[G-14-01\] `Market` struct could packed more strictly](#g-)
  - [\[G-15\] `storage` can be packed more precisely](#g-)
  - [\[G-16\] `constant` should be value not `expressions`](#g-)


## [G-01]Using immutable on variables that are only set in the constructor and never after 

**Gas per instance: 2.1K**
**Total Instances: 7**
Total Gas Saved: `2.1 * 7 = 14700 Gas`



```solidity
function initialize(
        address _xvsVault,
        address _xvsVaultRewardToken,
        uint256 _xvsVaultPoolId,
        uint128 _alphaNumerator,
        uint128 _alphaDenominator,
        address _accessControlManager,
        address _protocolShareReserve,
        address _primeLiquidityProvider,
        address _comptroller,
        address _oracle,
        uint256 _loopsLimit
    ) external virtual initializer {
        .......
        .......

        alphaNumerator = _alphaNumerator;
        alphaDenominator = _alphaDenominator;
        xvsVaultRewardToken = _xvsVaultRewardToken; // @audit immutable
        xvsVaultPoolId = _xvsVaultPoolId;  // @audit immu
        xvsVault = _xvsVault;  // @audit immu
        nextScoreUpdateRoundId = 0;  // @audit default
        protocolShareReserve = _protocolShareReserve;  // @audit immu
        primeLiquidityProvider = _primeLiquidityProvider;  // @audit imm
        comptroller = _comptroller;  // @audit imm
        oracle = ResilientOracleInterface(_oracle);  // @audit imm
```
Those state variable present in `PrimeStorage.sol` contract which will made immutables.



## [G-02] Variable should created outside of loops

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L179-L181
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol
        for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            uint256 interestAccrued = getInterestAccrued(market, user);
            uint256 accrued = interests[market][user].accrued;
```

```diff
+       address market;
+       uint256 interestAccrued;
+       uint256 accrued;

        for (uint256 i = 0; i < _allMarkets.length; ) {
+           market = _allMarkets[i];
+           interestAccrued = getInterestAccrued(market, user);
+           accrued = interests[market][user].accrued;
```

## [Gas-03] `address[] storage _allMarkets` could made as `calldata` instead `storage` in multiple instances
 If the data called to function does not need to be changed (like updating values in an array), it can be passed in as calldata.

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L175
```solidity
File: contracts/Tokens/Prime/Prime.sol
            function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
        address[] storage _allMarkets = allMarkets;
```
```diff
    function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
-       address[] storage _allMarkets = allMarkets;
+       address[] calldata _allMarkets = allMarkets;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L608

```diff
    function _accrueInterestAndUpdateScore(address user) internal {
-       address[] storage _allMarkets = allMarkets;
+       address[] calldata _allMarkets = allMarkets;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L728

```diff
function _burn(address user) internal {
        if (!tokens[user].exists) revert UserHasNoPrimeToken();
-       address[] storage _allMarkets = allMarkets;
+       address[] calldata _allMarkets = allMarkets;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L608

```diff
    function _accrueInterestAndUpdateScore(address user) internal {
-       address[] storage _allMarkets = allMarkets;
+       address[] calldata _allMarkets = allMarkets;
```

## [Gas-04] Some State Variable could cached in `memory` with out calling them repeatedly (Those are missed by automated finding reports)

### [Gas-04-01] `nextScoreUpdateRoundId` could cached in memory as it called 3 times via `updateScores()` function

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230

```diff
  function updateScores(address[] memory users) external {
+       uint256 _nextScoreUpdateRoundId = nextScoreUpdateRoundId;
        if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
-       if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();
+       if (_nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

        for (uint256 i = 0; i < users.length; ) {
            address user = users[i];

            if (!tokens[user].exists) revert UserHasNoPrimeToken();
-           if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;
+           if (isScoreUpdated[_nextScoreUpdateRoundId][user]) continue;

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
-           isScoreUpdated[nextScoreUpdateRoundId][user] = true;
+           isScoreUpdated[_nextScoreUpdateRoundId][user] = true;

            unchecked {
                i++;
            }

            emit UserScoreUpdated(user);
        }
    }
```

## [Gas-05] If()/require() checks which make state variable or important variable checks should be at top of function.

### [Gas-05-01] `_ensureMaxLoops(allMarkets.length)` which checks all created markets are in limit range or not should at top of `addMarket()` 

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L288-L309
```diff
function addMarket(address vToken, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
        _checkAccessAllowed("addMarket(address,uint256,uint256)");
        if (markets[vToken].exists) revert MarketAlreadyExists();
+       _ensureMaxLoops(allMarkets.length + 1);

        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);
        if (!isMarketExist) revert InvalidVToken();

        markets[vToken].rewardIndex = 0;
        markets[vToken].supplyMultiplier = supplyMultiplier;
        markets[vToken].borrowMultiplier = borrowMultiplier;
        markets[vToken].sumOfMembersScore = 0;
        markets[vToken].exists = true;

        vTokenForAsset[_getUnderlying(vToken)] = vToken;

        allMarkets.push(vToken);
        _startScoreUpdateRound();

-       _ensureMaxLoops(allMarkets.length);

        emit MarketAdded(vToken, supplyMultiplier, borrowMultiplier);
    }

```

## [Gas-06] To update one limit (irrevocableLimit or revocableLimit) other automatically get override

Each time one limit change other will have to chage(or re-right with same value again), this will increase one extra wright to storage even if this behaviour was not intended.
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L316-324
```diff
    function setLimit(uint256 _irrevocableLimit, uint256 _revocableLimit) external {
        _checkAccessAllowed("setLimit(uint256,uint256)");
        if (_irrevocableLimit < totalIrrevocable || _revocableLimit < totalRevocable) revert InvalidLimit();

        emit MintLimitsUpdated(irrevocableLimit, revocableLimit, _irrevocableLimit, _revocableLimit);

        revocableLimit = _revocableLimit;
        irrevocableLimit = _irrevocableLimit;
    }
```

## [Gas-07] Use `delete` key word instead of setting mapping variables to default

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L375-L376
```diff
        } else if (!isAccountEligible && !tokens[user].exists && stakedAt[user] > 0) {
-           stakedAt[user] = 0;
+           delete stakedAt[user];
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L460
```diff
        IVToken market = IVToken(vToken);
-       unreleasedPSRIncome[_getUnderlying(address(market))] = 0;
+       delete unreleasedPSRIncome[_getUnderlying(address(market))];
```

## [Gas-08] Use assembly for some more check work can save some extra gas
### [Gas-08-01] Use to check `msg.sender`

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L453
```solidity
    function updateAssetsState(address _comptroller, address asset) external {
        if (msg.sender != protocolShareReserve) revert InvalidCaller();
        if (comptroller != _comptroller) revert InvalidComptroller();
```
### [Gas-08-02] Use to check against some `state variables`
inputed `_comptroller` parameter check against `comptroller` stored in state variable, this could preform using assembly

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L454
```solidity
    function updateAssetsState(address _comptroller, address asset) external {
        if (msg.sender != protocolShareReserve) revert InvalidCaller();
        if (comptroller != _comptroller) revert InvalidComptroller();
```

## [Gas-9] Ternary Operator could used when ever possible

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L481-L485
```diff
        uint256 totalTimeStaked = block.timestamp - stakedAt[user];
+       (totalTimeStaked < STAKING_PERIOD) ? STAKING_PERIOD - totalTimeStaked : 0;

-       if (totalTimeStaked < STAKING_PERIOD) {
-           return STAKING_PERIOD - totalTimeStaked;
-       } else {
-           return 0;
-       }
```

## [Gas-10] Some Mapping value could be cached into `storage`

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L629-L633
```diff
    function _initializeMarkets(address account) internal {
        address[] storage _allMarkets = allMarkets;
        for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            accrueInterest(market);

+           Interset storage _interset = interests[market][account];
-           interests[market][account].rewardIndex = markets[market].rewardIndex;
+           _interset.rewardIndex = markets[market].rewardIndex;

            uint256 score = _calculateScore(market, account);
-           interests[market][account].score = score;
+           _interset.score = score;
            markets[market].sumOfMembersScore = markets[market].sumOfMembersScore + score;

            unchecked {
                i++;
            }
        }
    }
```

## [Gas-11] A extra storage call will be save by caching mathematical operation in memory 

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L818-L822
```solidity
    function _startScoreUpdateRound() internal {
        nextScoreUpdateRoundId++;
        totalScoreUpdatesRequired = totalIrrevocable + totalRevocable;
        pendingScoreUpdates = totalScoreUpdatesRequired;
    }
```
```diff
    function _startScoreUpdateRound() internal {
        nextScoreUpdateRoundId++;
+       uint256 _totalScoreUpdatesRequired = totalIrrevocable + totalRevocable;
-       totalScoreUpdatesRequired = totalIrrevocable + totalRevocable;
+       totalScoreUpdatesRequired = _totalScoreUpdatesRequired;
+       pendingScoreUpdates = _totalScoreUpdatesRequired;
-       pendingScoreUpdates = totalScoreUpdatesRequired;
    }
```

## [Gas-12] Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement (These are missing in automated finding reports)

### [Gas-12-01] `totalScoreUpdatesRequired--` could be uncheked because of previous `if (totalScoreUpdatesRequired > 0)` check
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L828
```diff
    function _updateRoundAfterTokenBurned(address user) internal {
-       if (totalScoreUpdatesRequired > 0) totalScoreUpdatesRequired--;
+       if (totalScoreUpdatesRequired > 0) unchecked{ totalScoreUpdatesRequired--;}
```
## [Gas-12-02] Due to `pendingScoreUpdates > 0` check `pendingScoreUpdates--` could be unchecked
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L831
```diff
        if (pendingScoreUpdates > 0 && !isScoreUpdated[nextScoreUpdateRoundId][user]) { 
-           pendingScoreUpdates--;
+           unchecked{ pendingScoreUpdates--; }
```

## [Gas-13] `address(this)` could cached in a `immutable` state variable

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L564
```diff
uint256 totalIncomeUnreleased = IProtocolShareReserve(protocolShareReserve).getUnreleasedFunds(
            comptroller,
            IProtocolShareReserve.Schema.SPREAD_PRIME_CORE,
            address(this),
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L682
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L686
```diff
        if (amount > asset.balanceOf(address(this))) {
            address[] memory assets = new address[](1);
            assets[0] = address(asset);
            IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
            if (amount > asset.balanceOf(address(this))) {
                IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
                unreleasedPLPIncome[underlying] = 0;
            }
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L960
```diff
          return
            IProtocolShareReserve(protocolShareReserve).getPercentageDistribution(
                address(this),
```

## [Gas-14] `struct` can be packed more precisely
### [Gas-14-01] `Market` struct could packed more strictly
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L
```solidity
    struct Market {
        uint256 supplyMultiplier;
        uint256 borrowMultiplier;
        uint256 rewardIndex;
        uint256 sumOfMembersScore;
        bool exists;
    }
```
```diff
    struct Market {
-       uint256 supplyMultiplier;
-       uint256 borrowMultiplier;
-       uint256 rewardIndex;
-       uint256 sumOfMembersScore;
-       bool exists;
 
+       uint128 supplyMultiplier;
+       uint128 borrowMultiplier;
+       uint120 rewardIndex;
+       uint128 sumOfMembersScore;
+       bool exists;
    }
```

## [Gas-15] `storage` can be packed more precisely

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L49-L58
```diff
-   uint256 public totalIrrevocable;

-   uint256 public totalRevocable;

-   uint256 public revocableLimit;

-   uint256 public irrevocableLimit;

+   uint128 public totalIrrevocable;

+   uint128 public totalRevocable;

+   uint128 public revocableLimit;

+   uint128 public irrevocableLimit;
```

## [Gas-16] `caonstant` should be value not `expressions`

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L
```diff
    /// @notice minimum amount of XVS user needs to stake to become a prime member
-   uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;
+   uint256 public constant MINIMUM_STAKED_XVS = [Resultant Value here];     // 1000 * EXP_SCALE => comment signifies expression

    /// @notice maximum XVS taken in account when calculating user score
    uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

    /// @notice number of days user need to stake to claim prime token
    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;
```
