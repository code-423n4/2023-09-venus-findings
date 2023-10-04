
### Low Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[L-1] Missing gap Storage Variable in Upgradeable Contract | [Missing gap Storage Variable in Upgradeable Contract](#missing-gap-storage-variable-in-upgradeable-contract) | 1 |
|[L-2] Reentrancy vulnerabilities | [Reentrancy vulnerabilities](#reentrancy-vulnerabilities) | 6 |
|[L-3] Avoid Usage of Ownable and Prefer Ownable2Step | [Avoid Usage of Ownable and Prefer Ownable2Step](#avoid-usage-of-ownable-and-prefer-ownable2step) | 2 |

Total: issues 3

### Non-Critical Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[N-1] Do not calculate constants | [Do not calculate constants](#do-not-calculate-constants) | 3 |
|[N-2] Costly operations inside a loop | [Costly operations inside a loop](#costly-operations-inside-a-loop) | 6 |
|[N-3] Missing events in sensitive functions | [Missing events in sensitive functions](#missing-events-in-sensitive-functions) | 3 |
|[N-4] Functions Not Implementing an Interface | [Functions Not Implementing an Interface](#functions-not-implementing-an-interface) | 32 |
|[N-5] Consider Disable Ownership Renouncement in Ownable Contracts | [Consider Disable Ownership Renouncement in Ownable Contracts](#consider-disable-ownership-renouncement-in-ownable-contracts) | 2 |
|[N-6] Event Emission Preceding External Calls: A Best Practice | [Event Emission Preceding External Calls: A Best Practice](#event-emission-preceding-external-calls-a-best-practice) | 6 |
|[N-7] Functions that alter state should emit events | [Functions that alter state should emit events](#functions-that-alter-state-should-emit-events) | 6 |
|[N-8] Too many digits | [Too many digits](#too-many-digits) | 69 |
|[N-9] Unused imports | [Unused imports](#unused-imports) | 2 |
|[N-10] Unused return | [Unused return](#unused-return) | 1 |
|[N-1] Whitespace in Expressions | [Whitespace in Expressions](#whitespace-in-expressions) | 2 |

Total: 11 issues


## Missing gap Storage Variable in Upgradeable Contract
- Severity: Low
- Confidence: High

### Note 
There is one instance that was missing in the wining bot. 

### Description
upgradeable contracts that are missing a '__gap' storage variable. In upgradeable contracts, it is important to reserve storage slots for future versions to introduce new storage variables. The '__gap' storage variable acts as a placeholder, allowing for seamless upgrades without affecting existing storage layout.When a contract is not designed with a '__gap' storage variable, adding new storage variables in subsequent versions becomes problematic. It can lead to storage collisions or layout incompatibilities, making it difficult to upgrade the contract without requiring costly data migrations or redeployments.

<details>

<summary>
There are 1 instance of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

35    contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, MaxLoopsLimitHelper, PrimeStorageV1 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L35-L1015](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L35-L1015)

</details>

# 



## Reentrancy vulnerabilities
- Severity: Low
- Confidence: Medium

### Description

Detection of the [reentrancy bug](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy).
Only report reentrancy that acts as a double call (see `reentrancy-eth`, `reentrancy-no-eth`).

<details>

<summary>
There are 6 instances of this issue:

</summary>

###
#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

725    function _burn(address user) internal 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

731    _executeBoost(user, _allMarkets[i])
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

570    _primeLiquidityProvider.accrueTokens(underlying)
```

	State variables written after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

753    _updateRoundAfterTokenBurned(user)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

831    pendingScoreUpdates--
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

745    totalIrrevocable--
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

747    totalRevocable--
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

753    _updateRoundAfterTokenBurned(user)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

828    totalScoreUpdatesRequired--
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L725-L756](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L725-L756)


#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

779    function _executeBoost(address user, address vToken) internal 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

784    accrueInterest(vToken)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

570    _primeLiquidityProvider.accrueTokens(underlying)
```

	State variables written after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

785    interests[vToken][user].accrued += _interestAccrued(vToken, user)
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

786    interests[vToken][user].rewardIndex = markets[vToken].rewardIndex
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L779-L787](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L779-L787)


#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

794    function _updateScore(address user, address market) internal 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

799    uint256 score = _calculateScore(market, user)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

657    oracle.updateAssetPrice(xvsToken)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

658    oracle.updatePrice(market)
```

	State variables written after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

801    interests[market][user].score = score
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L794-L802](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L794-L802)


#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

554    function accrueInterest(address vToken) public 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

570    _primeLiquidityProvider.accrueTokens(underlying)
```

	State variables written after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

581    unreleasedPLPIncome[underlying] = totalAccruedInPLP
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L554-L589](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L554-L589)


#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

237    function updateAlpha(uint128 _alphaNumerator, uint128 _alphaDenominator) external 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

247    accrueInterest(allMarkets[i])
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

570    _primeLiquidityProvider.accrueTokens(underlying)
```

	State variables written after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

254    _startScoreUpdateRound()
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

819    nextScoreUpdateRoundId++
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

254    _startScoreUpdateRound()
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

821    pendingScoreUpdates = totalScoreUpdatesRequired
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

254    _startScoreUpdateRound()
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

820    totalScoreUpdatesRequired = totalIrrevocable + totalRevocable
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L237-L255](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L237-L255)


#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

263    function updateMultipliers(address market, uint256 supplyMultiplier, uint256 borrowMultiplier) external 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

267    accrueInterest(market)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

570    _primeLiquidityProvider.accrueTokens(underlying)
```

	State variables written after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

279    _startScoreUpdateRound()
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

819    nextScoreUpdateRoundId++
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

279    _startScoreUpdateRound()
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

821    pendingScoreUpdates = totalScoreUpdatesRequired
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

279    _startScoreUpdateRound()
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

820    totalScoreUpdatesRequired = totalIrrevocable + totalRevocable
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L263-L280](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L263-L280)


</details>

# 

## Avoid Usage of Ownable and Prefer Ownable2Step
- Severity: Low
- Confidence: High

### Description
The 'Ownable'/'OwnableUpgradeable' contracts provides a basic ownership mechanism, but it lacks an additional step for secure ownership transfer, which is crucial for certain scenarios, such as upgrading contracts or handling complex ownership transitions.

By utilizing the 'Ownable2Step'/'Ownable2StepUpgradeable' contract, which extends the functionality of 'Ownable'/'OwnableUpgradeable' with a two-step ownership transfer process, developers can enhance the security and flexibility of their contracts. The two-step process involves the current owner initiating a transfer request, which requires confirmation from the new owner before the ownership is transferred.

Using 'Ownable2Step'/'Ownable2StepUpgradeable' mitigates risks associated with accidental or unauthorized ownership transfers, ensuring that ownership transitions are intentional and verified. It provides an extra layer of protection, particularly in situations where contract upgrades or complex ownership management are involved.

By leveraging 'Ownable2Step'/'Ownable2StepUpgradeable', developers can enhance the security and integrity of their contracts, promoting trust and reducing the potential for ownership-related vulnerabilities.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

35    contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, MaxLoopsLimitHelper, PrimeStorageV1 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L35-L1015](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L35-L1015)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

8    contract PrimeLiquidityProvider is AccessControlledV8, PausableUpgradeable 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L8-L349](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L8-L349)

</details>

# 



## Unused imports
- Severity: Non-Critical
- Confidence: High

### Description
Identify unused imports in source files that can be safely removed. Please note that this detector does not support files with cyclic imports or files that use 'import {...} from' directives.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#
Unused imports found in /Users/noam/Documents/Code4rena/2023-09-venus/contracts/Tokens/Prime/Prime.sol.
Consider removing the following imports:

    - 
```
File: contracts/Tokens/Prime/Prime.sol

18    import { InterfaceComptroller } from "./Interfaces/InterfaceComptroller.sol";
```
@venusprotocol/oracle/contracts/interfaces/OracleInterface.sol 
@venusprotocol/oracle/contracts/interfaces/OracleInterface.sol

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L18](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L18)


#
Unused imports found in /Users/noam/Documents/Code4rena/2023-09-venus/contracts/Tokens/Prime/libs/Scores.sol.
Consider removing the following imports:

    - 
```
File: contracts/Tokens/Prime/libs/Scores.sol

6    import { FixedMath } from "./FixedMath.sol";
```
@openzeppelin/contracts-upgradeable/utils/math/SafeCastUpgradeable.sol 
@openzeppelin/contracts-upgradeable/utils/math/SafeCastUpgradeable.sol

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/Scores.sol#L6](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/Scores.sol#L6)


</details>

# 


## Unused return
- Severity: Non-Critical
- Confidence: Medium

### Description
The return value of an external call is not stored in a local or state variable.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

841    (uint256 xvs, , uint256 pendingWithdrawals) = IXVSVault(xvsVault).getUserInfo(
842                xvsVaultRewardToken,
843                xvsVaultPoolId,
844                user
845            )
```
 ignores return value 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L841-L845](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L841-L845)


</details>

# 



## Whitespace in Expressions
- Severity: Non-Critical
- Confidence: High

### Description
Detects when whitespace usage in expressions does not conform to the Solidity style guide.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

35    contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, MaxLoopsLimitHelper, PrimeStorageV1 
```

```
// @audit: whitespace inside parenthesis
Line: 156            for (uint256 i = 0; i < _allMarkets.length; ) {


// @audit: whitespace inside parenthesis
Line: 179            for (uint256 i = 0; i < users.length; ) {


// @audit: whitespace inside parenthesis
Line: 186                for (uint256 j = 0; j < _allMarkets.length; ) {


// @audit: whitespace inside parenthesis
Line: 217            for (uint256 i = 0; i < allMarkets.length; ) {


// @audit: whitespace inside parenthesis
Line: 288                for (uint256 i = 0; i < users.length; ) {


// @audit: whitespace inside parenthesis
Line: 302                for (uint256 i = 0; i < users.length; ) {


// @audit: whitespace inside parenthesis
Line: 502            for (uint256 i = 0; i < _allMarkets.length; ) {


// @audit: whitespace inside parenthesis
Line: 515            for (uint256 i = 0; i < _allMarkets.length; ) {


// @audit: whitespace inside parenthesis
Line: 545            (uint256 capital, , ) = _capitalForScore(xvsBalanceForScore, borrow, supply, market);


// @audit: whitespace inside parenthesis
Line: 603            for (uint256 i = 0; i < _allMarkets.length; ) {


```
[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L35-L1015](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L35-L1015)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

8    contract PrimeLiquidityProvider is AccessControlledV8, PausableUpgradeable 
```

```
// @audit: whitespace inside parenthesis
Line: 96            for (uint256 i; i < numTokens; ) {


// @audit: whitespace inside parenthesis
Line: 108            for (uint256 i; i < tokens_.length; ) {


// @audit: whitespace inside parenthesis
Line: 138            for (uint256 i; i < numTokens; ) {


```
[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L8-L349](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L8-L349)


</details>

# 



## Do not calculate constants
- Severity: Non-Critical
- Confidence: High

### Description
Due to how constant variables are implemented in Solidity (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas. While in most cases, the compiler will optimize these computations away, it is considered a best practice to write code that does not rely on the compiler optimization.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/PrimeStorage.sol

34    uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L34](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L34)


#

```
File: contracts/Tokens/Prime/PrimeStorage.sol

37    uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L37](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L37)


#

```
File: contracts/Tokens/Prime/PrimeStorage.sol

40    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L40](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L40)


</details>

# 


## Costly operations inside a loop
- Severity: Non-Critical
- Confidence: Medium

### Description
Costly operations inside a loop might waste gas, so optimizations are justified.

<details>

<summary>
There are 6 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

200    function updateScores(address[] memory users) external 
```
 has costly operations inside a loop:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

221    pendingScoreUpdates--
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230)


#

```
File: contracts/Tokens/Prime/Prime.sol

762    function _upgrade(address user) internal 
```
 has costly operations inside a loop:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

766    totalIrrevocable++
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L762-L772](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L762-L772)


#

```
File: contracts/Tokens/Prime/Prime.sol

762    function _upgrade(address user) internal 
```
 has costly operations inside a loop:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

767    totalRevocable--
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L762-L772](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L762-L772)


#

```
File: contracts/Tokens/Prime/Prime.sol

704    function _mint(bool isIrrevocable, address user) internal 
```
 has costly operations inside a loop:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

711    totalIrrevocable++
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L704-L719](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L704-L719)


#

```
File: contracts/Tokens/Prime/Prime.sol

704    function _mint(bool isIrrevocable, address user) internal 
```
 has costly operations inside a loop:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

713    totalRevocable++
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L704-L719](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L704-L719)


#

```
File: contracts/Tokens/Prime/Prime.sol

331    function issue(bool isIrrevocable, address[] calldata users) external 
```
 has costly operations inside a loop:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

352    delete stakedAt[users[i]]
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359)


</details>

# 



## Missing events in sensitive functions
- Severity: Non-Critical
- Confidence: High

### Description
Events should be emitted when sensitive changes are made to the contracts, but some functions lack them. 

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

794    function _updateScore(address user, address market) internal 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L794-L802](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L794-L802)


#

```
File: contracts/Tokens/Prime/Prime.sol

818    function _startScoreUpdateRound() internal 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L818-L822](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L818-L822)


#

```
File: contracts/Tokens/Prime/Prime.sol

827    function _updateRoundAfterTokenBurned(address user) internal 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L827-L833](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L827-L833)


</details>

# 


## Consider Disable Ownership Renouncement in Ownable Contracts
- Severity: Non-Critical
- Confidence: High

### Description
Ownership renouncement is a feature provided by the Ownable contract in various smart contract frameworks, such as OpenZeppelin. It allows the contract owner to transfer ownership to another address, relinquishing their control over the contract. However, in certain cases, it may be undesirable or unnecessary to allow ownership renouncement. 

It is important to carefully consider the implications of allowing ownership renouncement. Disabling renouncement can provide additional security and prevent potential risks associated with unintended ownership transfers.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

35    contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, MaxLoopsLimitHelper, PrimeStorageV1 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L35-L1015](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L35-L1015)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

8    contract PrimeLiquidityProvider is AccessControlledV8, PausableUpgradeable 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L8-L349](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L8-L349)


</details>

# 


## Functions that alter state should emit events
- Severity: Non-Critical
- Confidence: Medium

### Description
Functions that alter the state of the contract should emit an event to inform external observers of the change.

<details>

<summary>
There are 6 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

554    function accrueInterest(address vToken) public 
```
 The function `accrueInterest` changes state but does not emit an event.

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L554-L589](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L554-L589)


#

```
File: contracts/Tokens/Prime/Prime.sol

623    function _initializeMarkets(address account) internal 
```
 The function `_initializeMarkets` changes state but does not emit an event.

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L623-L639](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L623-L639)


#

```
File: contracts/Tokens/Prime/Prime.sol

779    function _executeBoost(address user, address vToken) internal 
```
 The function `_executeBoost` changes state but does not emit an event.

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L779-L787](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L779-L787)


#

```
File: contracts/Tokens/Prime/Prime.sol

794    function _updateScore(address user, address market) internal 
```
 The function `_updateScore` changes state but does not emit an event.

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L794-L802](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L794-L802)


#

```
File: contracts/Tokens/Prime/Prime.sol

818    function _startScoreUpdateRound() internal 
```
 The function `_startScoreUpdateRound` changes state but does not emit an event.

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L818-L822](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L818-L822)


#

```
File: contracts/Tokens/Prime/Prime.sol

827    function _updateRoundAfterTokenBurned(address user) internal 
```
 The function `_updateRoundAfterTokenBurned` changes state but does not emit an event.

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L827-L833](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L827-L833)


</details>

# 


## Too many digits
- Severity: Non-Critical
- Confidence: Medium

### Description

Literals with many digits are difficult to read and review.


<details>

<summary>
There are 69 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

73    x <= int256(0x00000000000000000000000000000000000000000001c8464f76164760000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

74    r -= int256(0x0000000000000000000000000000001000000000000000000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

75    x = (x * FIXED_1) / int256(0x00000000000000000000000000000000000000000001c8464f76164760000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

78    x <= int256(0x00000000000000000000000000000000000000f1aaddd7742e90000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

79    r -= int256(0x0000000000000000000000000000000800000000000000000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

80    x = (x * FIXED_1) / int256(0x00000000000000000000000000000000000000f1aaddd7742e90000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

83    x <= int256(0x00000000000000000000000000000000000afe10820813d78000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

84    r -= int256(0x0000000000000000000000000000000400000000000000000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

85    x = (x * FIXED_1) / int256(0x00000000000000000000000000000000000afe10820813d78000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

88    x <= int256(0x0000000000000000000000000000000002582ab704279ec00000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

89    r -= int256(0x0000000000000000000000000000000200000000000000000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

90    x = (x * FIXED_1) / int256(0x0000000000000000000000000000000002582ab704279ec00000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

93    x <= int256(0x000000000000000000000000000000001152aaa3bf81cc000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

94    r -= int256(0x0000000000000000000000000000000100000000000000000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

95    x = (x * FIXED_1) / int256(0x000000000000000000000000000000001152aaa3bf81cc000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

98    x <= int256(0x000000000000000000000000000000002f16ac6c59de70000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

99    r -= int256(0x0000000000000000000000000000000080000000000000000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

100    x = (x * FIXED_1) / int256(0x000000000000000000000000000000002f16ac6c59de70000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

103    x <= int256(0x000000000000000000000000000000004da2cbf1be5828000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

104    r -= int256(0x0000000000000000000000000000000040000000000000000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

105    x = (x * FIXED_1) / int256(0x000000000000000000000000000000004da2cbf1be5828000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

108    x <= int256(0x0000000000000000000000000000000063afbe7ab2082c000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

109    r -= int256(0x0000000000000000000000000000000020000000000000000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

110    x = (x * FIXED_1) / int256(0x0000000000000000000000000000000063afbe7ab2082c000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

113    x <= int256(0x0000000000000000000000000000000070f5a893b608861e1f58934f97aea57d)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

114    r -= int256(0x0000000000000000000000000000000010000000000000000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

115    x = (x * FIXED_1) / int256(0x0000000000000000000000000000000070f5a893b608861e1f58934f97aea57d)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

122    r += (z * (0x100000000000000000000000000000000 - y)) / 0x100000000000000000000000000000000
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

124    r += (z * (0x0aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa - y)) / 0x200000000000000000000000000000000
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

126    r += (z * (0x099999999999999999999999999999999 - y)) / 0x300000000000000000000000000000000
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

128    r += (z * (0x092492492492492492492492492492492 - y)) / 0x400000000000000000000000000000000
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

130    r += (z * (0x08e38e38e38e38e38e38e38e38e38e38e - y)) / 0x500000000000000000000000000000000
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

132    r += (z * (0x08ba2e8ba2e8ba2e8ba2e8ba2e8ba2e8b - y)) / 0x600000000000000000000000000000000
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

134    r += (z * (0x089d89d89d89d89d89d89d89d89d89d89 - y)) / 0x700000000000000000000000000000000
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51    function ln(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

136    r += (z * (0x088888888888888888888888888888888 - y)) / 0x800000000000000000000000000000000
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L51-L137)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

162    z = y = x % 0x0000000000000000000000000000000010000000000000000000000000000000
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

178    r += z * 0x00000618fee9f800
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

180    r += z * 0x0000009c197dcc00
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

182    r += z * 0x0000000e30dce400
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

184    r += z * 0x000000012ebd1300
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

186    r += z * 0x0000000017499f00
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

188    r += z * 0x0000000001a9d480
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

190    r += z * 0x00000000001c6380
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

192    r += z * 0x000000000001c638
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

194    r += z * 0x0000000000001ab8
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

196    r += z * 0x000000000000017c
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

198    r += z * 0x0000000000000014
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

200    r += z * 0x0000000000000001
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

206    (x & int256(0x0000000000000000000000000000001000000000000000000000000000000000)) != 0
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

207    r =
208                    (r * int256(0x00000000000000000000000000000000000000f1aaddd7742e56d32fb9f99744)) /
209                    int256(0x0000000000000000000000000043cbaf42a000812488fc5c220ad7b97bf6e99e)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

212    (x & int256(0x0000000000000000000000000000000800000000000000000000000000000000)) != 0
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

213    r =
214                    (r * int256(0x00000000000000000000000000000000000afe10820813d65dfe6a33c07f738f)) /
215                    int256(0x000000000000000000000000000005d27a9f51c31b7c2f8038212a0574779991)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

218    (x & int256(0x0000000000000000000000000000000400000000000000000000000000000000)) != 0
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

219    r =
220                    (r * int256(0x0000000000000000000000000000000002582ab704279e8efd15e0265855c47a)) /
221                    int256(0x0000000000000000000000000000001b4c902e273a58678d6d3bfdb93db96d02)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

224    (x & int256(0x0000000000000000000000000000000200000000000000000000000000000000)) != 0
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

225    r =
226                    (r * int256(0x000000000000000000000000000000001152aaa3bf81cb9fdb76eae12d029571)) /
227                    int256(0x00000000000000000000000000000003b1cc971a9bb5b9867477440d6d157750)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

230    (x & int256(0x0000000000000000000000000000000100000000000000000000000000000000)) != 0
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

231    r =
232                    (r * int256(0x000000000000000000000000000000002f16ac6c59de6f8d5d6f63c1482a7c86)) /
233                    int256(0x000000000000000000000000000000015bf0a8b1457695355fb8ac404e7a79e3)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

236    (x & int256(0x0000000000000000000000000000000080000000000000000000000000000000)) != 0
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

237    r =
238                    (r * int256(0x000000000000000000000000000000004da2cbf1be5827f9eb3ad1aa9866ebb3)) /
239                    int256(0x00000000000000000000000000000000d3094c70f034de4b96ff7d5b6f99fcd8)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

242    (x & int256(0x0000000000000000000000000000000040000000000000000000000000000000)) != 0
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

243    r =
244                    (r * int256(0x0000000000000000000000000000000063afbe7ab2082ba1a0ae5e4eb1b479dc)) /
245                    int256(0x00000000000000000000000000000000a45af1e1f40c333b3de1db4dd55f29a7)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

248    (x & int256(0x0000000000000000000000000000000020000000000000000000000000000000)) != 0
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

249    r =
250                    (r * int256(0x0000000000000000000000000000000070f5a893b608861e1f58934f97aea57d)) /
251                    int256(0x00000000000000000000000000000000910b022db7ae67ce76b441c27035c6a1)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

254    (x & int256(0x0000000000000000000000000000000010000000000000000000000000000000)) != 0
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

140    function exp(int256 x) internal pure returns (int256 r) 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

255    r =
256                    (r * int256(0x00000000000000000000000000000000783eafef1c0a8f3978c7f81824d62ebf)) /
257                    int256(0x0000000000000000000000000000000088415abbe9a76bead8d00cf112e4d4a8)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L140-L259)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

38    library FixedMath0x 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

40    int256 internal constant FIXED_1 = int256(0x0000000000000000000000000000000080000000000000000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L38-L260](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L38-L260)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

38    library FixedMath0x 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

44    int256 private constant LN_MIN_VAL = int256(0x0000000000000000000000000000000000000000000000000000000733048c5a)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L38-L260](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L38-L260)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

38    library FixedMath0x 
```
 uses literals with too many digits:
	- 
```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

48    int256 private constant EXP_MIN_VAL = -int256(0x0000000000000000000000000000001ff0000000000000000000000000000000)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L38-L260](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L38-L260)


</details>

# 




## Functions Not Implementing an Interface
- Severity: Non-Critical
- Confidence: High

### Description
Contracts with public or external functions should typically implement an interface for clarity and modularity. If they do not, it could lead to issues with understanding the contract's intended API and maintainability concerns.

<details>

<summary>
There are 32 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

130    function initialize(
131            address _xvsVault,
132            address _xvsVaultRewardToken,
133            uint256 _xvsVaultPoolId,
134            uint128 _alphaNumerator,
135            uint128 _alphaDenominator,
136            address _accessControlManager,
137            address _protocolShareReserve,
138            address _primeLiquidityProvider,
139            address _comptroller,
140            address _oracle,
141            uint256 _loopsLimit
142        ) external virtual initializer 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L130-L167](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L130-L167)


#

```
File: contracts/Tokens/Prime/Prime.sol

174    function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L194](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L194)


#

```
File: contracts/Tokens/Prime/Prime.sol

200    function updateScores(address[] memory users) external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230)


#

```
File: contracts/Tokens/Prime/Prime.sol

237    function updateAlpha(uint128 _alphaNumerator, uint128 _alphaDenominator) external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L237-L255](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L237-L255)


#

```
File: contracts/Tokens/Prime/Prime.sol

263    function updateMultipliers(address market, uint256 supplyMultiplier, uint256 borrowMultiplier) external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L263-L280](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L263-L280)


#

```
File: contracts/Tokens/Prime/Prime.sol

288    function addMarket(address vToken, uint256 supplyMultiplier, uint256 borrowMultiplier) external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L288-L309](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L288-L309)


#

```
File: contracts/Tokens/Prime/Prime.sol

316    function setLimit(uint256 _irrevocableLimit, uint256 _revocableLimit) external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L316-L324](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L316-L324)


#

```
File: contracts/Tokens/Prime/Prime.sol

331    function issue(bool isIrrevocable, address[] calldata users) external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359)


#

```
File: contracts/Tokens/Prime/Prime.sol

365    function xvsUpdated(address user) external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L365-L382](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L365-L382)


#

```
File: contracts/Tokens/Prime/Prime.sol

389    function accrueInterestAndUpdateScore(address user, address market) external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L389-L392](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L389-L392)


#

```
File: contracts/Tokens/Prime/Prime.sol

397    function claim() external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L397-L405](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L397-L405)


#

```
File: contracts/Tokens/Prime/Prime.sol

411    function burn(address user) external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L411-L414](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L411-L414)


#

```
File: contracts/Tokens/Prime/Prime.sol

419    function togglePause() external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L419-L426](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L419-L426)


#

```
File: contracts/Tokens/Prime/Prime.sol

433    function claimInterest(address vToken) external whenNotPaused returns (uint256) 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L433-L435](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L433-L435)


#

```
File: contracts/Tokens/Prime/Prime.sol

443    function claimInterest(address vToken, address user) external whenNotPaused returns (uint256) 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L443-L445](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L443-L445)


#

```
File: contracts/Tokens/Prime/Prime.sol

469    function getAllMarkets() external view returns (address[] memory) 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L469-L471](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L469-L471)


#

```
File: contracts/Tokens/Prime/Prime.sol

478    function claimTimeRemaining(address user) external view returns (uint256) 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L478-L487](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L478-L487)


#

```
File: contracts/Tokens/Prime/Prime.sol

496    function calculateAPR(address market, address user) external view returns (uint256 supplyAPR, uint256 borrowAPR) 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496-L515](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496-L515)


#

```
File: contracts/Tokens/Prime/Prime.sol

527    function estimateAPR(
528            address market,
529            address user,
530            uint256 borrow,
531            uint256 supply,
532            uint256 xvsStaked
533        ) external view returns (uint256 supplyAPR, uint256 borrowAPR) 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548)


#

```
File: contracts/Tokens/Prime/Prime.sol

554    function accrueInterest(address vToken) public 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L554-L589](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L554-L589)


#

```
File: contracts/Tokens/Prime/Prime.sol

597    function getInterestAccrued(address vToken, address user) public returns (uint256) 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L597-L601](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L597-L601)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

90    function initialize(
91            address accessControlManager_,
92            address[] calldata tokens_,
93            uint256[] calldata distributionSpeeds_
94        ) external initializer 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L90-L111](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L90-L111)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

118    function initializeTokens(address[] calldata tokens_) external onlyOwner 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118-L126](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118-L126)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

132    function pauseFundsTransfer() external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L132-L135](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L132-L135)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

141    function resumeFundsTransfer() external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L141-L144](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L141-L144)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

153    function setTokensDistributionSpeed(address[] calldata tokens_, uint256[] calldata distributionSpeeds_) external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L153-L169](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L153-L169)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

177    function setPrimeToken(address prime_) external onlyOwner 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177-L182](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177-L182)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

192    function releaseFunds(address token_) external 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192-L205](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192-L205)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

216    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

232    function getEffectiveDistributionSpeed(address token_) external view returns (uint256) 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L232-L242](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L232-L242)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

249    function accrueTokens(address token_) public 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249-L272](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249-L272)


#

```
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

276    function getBlockNumber() public view virtual returns (uint256) 
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L276-L278](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L276-L278)


</details>

# 


## Event Emission Preceding External Calls: A Best Practice
- Severity: Non-Critical
- Confidence: Medium

### Description

Ensure that events follow the best practice of check-effects-interaction, and are emitted before external calls


<details>

<summary>
There are 6 instances of this issue:

</summary>

###
#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

725    function _burn(address user) internal 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

731    _executeBoost(user, _allMarkets[i])
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

570    _primeLiquidityProvider.accrueTokens(underlying)
```

	Event emitted after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

755    emit Burn(user)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L725-L756](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L725-L756)


#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

672    function _claimInterest(address vToken, address user) internal returns (uint256) 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

673    uint256 amount = getInterestAccrued(vToken, user)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

570    _primeLiquidityProvider.accrueTokens(underlying)
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

685    IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets)
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

687    IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset))
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

692    asset.safeTransfer(user, amount)
```

	Event emitted after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

694    emit InterestClaimed(user, vToken, amount)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L672-L697](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L672-L697)


#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

331    function issue(bool isIrrevocable, address[] calldata users) external 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

341    _initializeMarkets(users[i])
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

657    oracle.updateAssetPrice(xvsToken)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

570    _primeLiquidityProvider.accrueTokens(underlying)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

658    oracle.updatePrice(market)
```

	Event emitted after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

718    emit Mint(user, isIrrevocable)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

340    _mint(true, users[i])
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

771    emit TokenUpgraded(user)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

338    _upgrade(users[i])
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359)


#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

331    function issue(bool isIrrevocable, address[] calldata users) external 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

351    _initializeMarkets(users[i])
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

657    oracle.updateAssetPrice(xvsToken)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

570    _primeLiquidityProvider.accrueTokens(underlying)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

658    oracle.updatePrice(market)
```

	Event emitted after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

718    emit Mint(user, isIrrevocable)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

350    _mint(false, users[i])
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359)


#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

263    function updateMultipliers(address market, uint256 supplyMultiplier, uint256 borrowMultiplier) external 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

267    accrueInterest(market)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

570    _primeLiquidityProvider.accrueTokens(underlying)
```

	Event emitted after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

269    emit MultiplierUpdated(
270                market,
271                markets[market].supplyMultiplier,
272                markets[market].borrowMultiplier,
273                supplyMultiplier,
274                borrowMultiplier
275            )
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L263-L280](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L263-L280)


#
Reentrancy in 
```
File: contracts/Tokens/Prime/Prime.sol

200    function updateScores(address[] memory users) external 
```
:
	External calls:
	- 
```
File: contracts/Tokens/Prime/Prime.sol

213    _executeBoost(user, market)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

570    _primeLiquidityProvider.accrueTokens(underlying)
```

	- 
```
File: contracts/Tokens/Prime/Prime.sol

214    _updateScore(user, market)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

657    oracle.updateAssetPrice(xvsToken)
```

		- 
```
File: contracts/Tokens/Prime/Prime.sol

658    oracle.updatePrice(market)
```

	Event emitted after the call(s):
	- 
```
File: contracts/Tokens/Prime/Prime.sol

228    emit UserScoreUpdated(user)
```


[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230)


</details>

# 