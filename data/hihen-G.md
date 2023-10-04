## Gas Optimizations

Total **43 instances** over **7 issues**with **20773** gas saved:

|ID|Issue|Instances|Gas|
|:--:|:---|:--:|:--:|
| [[G&#x2011;01]](#g01-use-shift-right-instead-of-division-if-possible) | Use shift right instead of division if possible | 4 | 80 |
| [[G&#x2011;02]](#g02-using-x--x-instead-of-xx---can-save-gas) | Using `++X/`--X` instead of `X++`/`X--` can save gas | 19 | 95 |
| [[G&#x2011;03]](#g03-do-not-cache-state-variables-that-are-used-only-once) | Do not cache state variables that are used only once | 9 | 27 |
| [[G&#x2011;04]](#g04-functions-that-revert-when-called-by-normal-users-can-be-marked-payable) | Functions that revert when called by normal users can be marked `payable` | 3 | 63 |
| [[G&#x2011;05]](#g05-use-x--y-instead-of-x--x--y-for-mapping-state-variables) | Use `x += y` instead of `x = x + y` for mapping state variables | 3 | 120 |
| [[G&#x2011;06]](#g06-contracts-can-use-fewer-storage-slots-by-truncating-state-variables) | Contracts can use fewer storage slots by truncating state variables | 1 | 20000 |
| [[G&#x2011;07]](#g07-state-variable-access-within-a-loop) | State variable access within a loop | 1 | 97 |

## Gas Optimizations

### [G&#x2011;01] Use shift right instead of division if possible


Shifting right by `n` is like dividing by `2^n`, while the shifting cost less gas because it does not require checks and jumps.

There are 4 instances:

- *FixedMath0x.sol* ( [#L122](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/libs/FixedMath0x.sol#L122), [#L124](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/libs/FixedMath0x.sol#L124), [#L128](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/libs/FixedMath0x.sol#L128), [#L136](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/libs/FixedMath0x.sol#L136) ):

```solidity
/// @audit div 0x100000000000000000000000000000000
122:         r += (z * (0x100000000000000000000000000000000 - y)) / 0x100000000000000000000000000000000;

/// @audit div 0x200000000000000000000000000000000
124:         r += (z * (0x0aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa - y)) / 0x200000000000000000000000000000000;

/// @audit div 0x400000000000000000000000000000000
128:         r += (z * (0x092492492492492492492492492492492 - y)) / 0x400000000000000000000000000000000;

/// @audit div 0x800000000000000000000000000000000
136:         r += (z * (0x088888888888888888888888888888888 - y)) / 0x800000000000000000000000000000000; // add y^15 / 15 - y^16 / 16
```

### [G&#x2011;02] Using `++X/`--X` instead of `X++`/`X--` can save gas


It can save 5 gas for each execution / per iteration.

There are 19 instances (click to show):

- *Prime.sol* ( [#L189](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L189), [#L217](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L217), [#L221](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L221), [#L225](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L225), [#L250](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L250), [#L345](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L345), [#L355](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L355), [#L614](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L614), [#L636](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L636), [#L711](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L711), [#L713](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L713), [#L740](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L740), [#L745](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L745), [#L747](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L747), [#L766](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L766), [#L767](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L767), [#L819](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L819), [#L828](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L828), [#L831](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L831) ):

```solidity
189:                 i++;

217:                     j++;

221:             pendingScoreUpdates--;

225:                 i++;

250:                 i++;

345:                     i++;

355:                     i++;

614:                 i++;

636:                 i++;

711:             totalIrrevocable++;

713:             totalRevocable++;

740:                 i++;

745:             totalIrrevocable--;

747:             totalRevocable--;

766:         totalIrrevocable++;

767:         totalRevocable--;

819:         nextScoreUpdateRoundId++;

828:         if (totalScoreUpdatesRequired > 0) totalScoreUpdatesRequired--;

831:             pendingScoreUpdates--;
```

### [G&#x2011;03] Do not cache state variables that are used only once


It's cheaper to access the state variable directly if it is accessed only once. This can save the 3 gas cost of the extra stack allocation.

There are 9 instances:

- *Prime.sol* ( [#L181](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L181), [#L503](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L503), [#L504](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L504), [#L763](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L763), [#L920](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L920) ):

```solidity
181:             uint256 accrued = interests[market][user].accrued;

503:         uint256 userScore = interests[market][user].score;

504:         uint256 totalScore = markets[market].sumOfMembersScore;

763:         Token storage userToken = tokens[user];

920:         uint256 score = interests[vToken][user].score;
```

- *PrimeLiquidityProvider.sol* ( [#L233](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L233), [#L235](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L235), [#L289](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L289), [#L333](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L333) ):

```solidity
233:         uint256 distributionSpeed = tokenDistributionSpeeds[token_];

235:         uint256 accrued = tokenAmountAccrued[token_];

289:         uint256 initializedBlock = lastAccruedBlock[token_];

333:         uint256 lastBlockAccrued = lastAccruedBlock[token_];
```

### [G&#x2011;04] Functions that revert when called by normal users can be marked `payable`


If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
The extra opcodes avoided are: 
`CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2)` 
which cost an average of about 21 gas per call to the function, in addition to the extra deployment cost.

There are 3 instances:

- *PrimeLiquidityProvider.sol* ( [#L118](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118), [#L177](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177), [#L216](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216) ):

```solidity
118:     function initializeTokens(address[] calldata tokens_) external onlyOwner {

177:     function setPrimeToken(address prime_) external onlyOwner {

216:     function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
```

### [G&#x2011;05] Use `x += y` instead of `x = x + y` for mapping state variables


Using `+=` for mappings saves **40 gas** due to not having to recalculate the mapping's value's hash. The same applies to `-=`, `*=`, etc.

There are 3 instances:

- *Prime.sol* ( [#L588](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L588), [#L633](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L633), [#L733-L735](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L733-L735) ):

```solidity
588:         markets[vToken].rewardIndex = markets[vToken].rewardIndex + delta;

633:             markets[market].sumOfMembersScore = markets[market].sumOfMembersScore + score;

733:             markets[_allMarkets[i]].sumOfMembersScore =
734:                 markets[_allMarkets[i]].sumOfMembersScore -
735:                 interests[_allMarkets[i]][user].score;
```

### [G&#x2011;06] Contracts can use fewer storage slots by truncating state variables


Some state variables can be safely modified so that the contract uses fewer storage slots. Each saved slot can avoid an extra Gsset (**20000 gas**) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There is 1 instance (click to show):

- *PrimeStorage.sol* ( [#L6](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/PrimeStorage.sol#L6) ):

```solidity
/// @audit Truncate `nextScoreUpdateRoundId` to `uint64`
/// @audit Fewer storage usage (49 slots -> 48 slots):
///        800 B: uint256[25] __gap
///        32 B: mapping(address => Token) tokens
///        32 B: uint256 totalIrrevocable
///        32 B: uint256 totalRevocable
///        32 B: uint256 revocableLimit
///        32 B: uint256 irrevocableLimit
///        32 B: mapping(address => uint256) stakedAt
///        32 B: mapping(address => Market) markets
///        32 B: mapping(address => mapping(address => Interest)) interests
///        32 B: address[] allMarkets
///        32 B: uint256 xvsVaultPoolId
///        32 B: mapping(uint256 => mapping(address => bool)) isScoreUpdated
///        32 B: uint256 totalScoreUpdatesRequired
///        32 B: uint256 pendingScoreUpdates
///        32 B: mapping(address => address) vTokenForAsset
///        32 B: mapping(address => uint256) unreleasedPSRIncome
///        32 B: mapping(address => uint256) unreleasedPLPIncome
///        20 B: address xvsVault
///        4 B: uint64 nextScoreUpdateRoundId
///        20 B: address xvsVaultRewardToken
///        20 B: address protocolShareReserve
///        20 B: address comptroller
///        20 B: address primeLiquidityProvider
///        20 B: ResilientOracleInterface oracle
///        16 B: uint128 alphaNumerator
///        16 B: uint128 alphaDenominator
6: contract PrimeStorageV1 {
```

### [G&#x2011;07] State variable access within a loop

State variable reads and writes are more expensive than local variable reads and writes. Therefore, it is recommended to replace state variable reads and writes within loops with a local variable. Gas savings should be multiplied by the average loop length.

There are 1 instances:

- *Prime.sol* ( [#L246](https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L246) ):

```solidity
/// @audit Access `allMarkets.length` within for loop.
246:         for (uint256 i = 0; i < allMarkets.length; ) {
```
