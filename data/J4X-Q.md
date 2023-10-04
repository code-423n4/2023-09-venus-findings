# Low

## [L-01] Revert Risk in User Score Update Process
[Prime: Line 200](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200)

**Issue Description:
The smart contract utilizes score update rounds when certain actions occur, such as adding a new market, setting a new alpha value, or updating multipliers. The process of an update round looks like this.
1. One of the 3 action gets taken which leads to a call to `_startScoreUpdateRound()`
2. `nextScoreUpdateRoundId` gets increased by one and `pendingScoreUpdates` gets set to the amount of tokens we have.
3. Now as there could be X amount of users the function `updateScores()` gets called multiple times to update each users score.
4. For each user it's checked if this user has already been updated in the current round, if he has the token and if there are still users that need to be updated (which can be checked by decreasing `pendingScoreUpdates` each time a user gets updated)

However, a potential issue arises when the script is executed twice in a short timeframe. In such cases, the first execution's calls to `updateScores()` may not have completed when the second execution begins, potentially causing a transaction revert due to an incomplete update process.

**Proof of Concept (POC):**
- Protocol has 3 users: A, B, C
- `updateScores()` can update a maximum of 2 users before running out of gas
- `nextScoreUpdateRoundId = 0`
- `pendingScoreUpdates = 0`

Actions:
1. A new market is added, triggering `_startScoreUpdateRound()`
    - `nextScoreUpdateRoundId = 1`
    - `pendingScoreUpdates = 3`
2. Script1 calls `updateUsers()` for Users A and B
    - `nextScoreUpdateRoundId = 1`
    - `pendingScoreUpdates = 1`
    - `isScoreUpdated[1][A, B] = true`
3. Multipliers are changed, triggering another `_startScoreUpdateRound()`
    - `nextScoreUpdateRoundId = 2`
    - `pendingScoreUpdates = 3`
4. Script1 calls `updateUsers()` for User C
    - `nextScoreUpdateRoundId = 2`
    - `pendingScoreUpdates = 2`
    - `isScoreUpdated[1][C] = true`
5. Script2 calls `updateUsers()` for Users A and B
    - `nextScoreUpdateRoundId = 2`
    - `pendingScoreUpdates = 0`
    - `isScoreUpdated[2][A, B] = true`
6. Script2 calls `updateUsers()` for User C
    - `nextScoreUpdateRoundId = 2`
    - `pendingScoreUpdates = 0`
    - Revert occurs due to the failed check `(pendingScoreUpdates == 0)`

**Recommended Mitigation Steps:**
To address this issue, a simple mitigation can be implemented. The contract should check if `pendingScoreUpdates` is not zero before allowing a new call to `_startScoreUpdateRound()`. This check can be added at the beginning of the `_startScoreUpdateRound()` function:

```solidity
if(pendingScoreUpdates != 0) revert;
```

Additionally, a custom error named `updatingRoundNotFinished` could be introduced to provide better clarity on the revert reason.

---
## [L-02] Unintended accrueTokens() Behavior on Arbitrum
[PrimeLiquidityProvider: Line 249](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249)

**Issue Description:**
The smart contract relies on `block.number` to perform certain operations, including the `accrueTokens()` function. However, on the Arbitrum network, `block.number` is updated only once per minute, resulting in the function accruing tokens approximately every 4 blocks instead of every block.

**Recommended Mitigation Steps:**
If accruing tokens once per minute aligns with the protocol's management requirements, it's advisable to retain the current implementation and update the documentation to reflect this behavior. However, if the protocol should accrue tokens on every block, consider using the `arbBlockNumber()` function from the [ArbSys](https://docs.arbitrum.io/for-devs/dev-tools-and-resources/precompiles#arbsys) to obtain the accurate block number.

---
## [L-03]  Incorrect return in claimTimeRemaining()
[Prime: Line 479](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L479)

**Issue Description:**
If a user has never staked, `(stakedAt[user] == 0)` the function returns the `STAKING_PERIOD` which is incorrect as this user has not staked and will not be able to retrieve anything when the staking period has passed. 

**Recommended Mitigation Steps:**
I would recommend returning `type(uint256).max` as this will probably make it more clear to a user that he has not staked enough.

---
## [L-04] Lack of Validation for Markets in calculateAPR()
[Prime: Line 496](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496)

**Issue Description:**
The `calculateAPR()` function, used to predict a user's APR on a provided market, does not validate whether the supplied address represents a valid market. Consequently, if an incorrect address is provided, the function may return arbitrary and inaccurate values.

**Recommended Mitigation Steps:**
To enhance the robustness of the `calculateAPR()` function, consider adding a validation check to ensure that the provided address corresponds to a valid prime market. If the address does not exist as a prime market, return 0 to prevent misleading data. A validation check could be implemented as follows:

```solidity
if (!markets[market].exists)
{
	return (0,0);
}
```

---
## [L-05] Incorrect return from getEffectiveDistributionSpeed()
[PrimeLiquidityProvider: Line 232](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L232)

**Issue Description:**
In case of the `accrueTokens()` function already having been called shortly before and leading to `IERC20Upgradeable(token_).balanceOf(address(PLP)) == tokenAmountAccrued[token_]`the `getEffectiveDistributionSpeed()` function incorrectly returns 0 which is not the distribution speed for the provided market. This not only results in an inaccurate calculation within Prime's `_incomeDistributionYearly()` function but also provides an incorrect return value to external callers of the view function.

**Recommended Mitigation Steps:**
To address this issue, it is advisable to modify the `getEffectiveDistributionSpeed()` function to always return `tokenDistributionSpeeds[token_]`, as follows:

```solidity
function getEffectiveDistributionSpeed(address token_) external view returns (uint256) {
	return tokenDistributionSpeeds[token_];
}
```

---


# Non-Critical (NC) Findings

## [NC-01] Missing Functions in IPrime Interface
[IPrime](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/IPrime.sol)

**Issue Description:**
The `IPrime` interface is lacking the inclusion of several external/public functions that are present in the `Prime` contract.

**Recommended Mitigation Steps:**
If the intention is to include all externally callable functions within the `IPrime` interface, it should be updated to include the missing functions listed below. However, if the interface is intended to expose functions only to users, the interface can exclude some functions.

Functions missing in `IPrime`:
- `initialize()`
- `getPendingInterests()`
- `updateScores()`
- `updateAlpha()`
- `updateMultipliers()`
- `addMarket()`
- `setLimit()`
- `issue()`
- `claim()`
- `burn()`
- `togglePause()`
- `2x claimInterest()`
- `updateAssetsState()`
- `getAllMarkets()`
- `claimTimeRemaining()`
- `calculateAPR()`
- `estimateAPR()`
- `getInterestAccrued()`

If only functions intended for the users should be included the interface is missing:
- `getPendingInterests()`
- `updateScores()`
- `claim()`
- `2x claimInterest()`
- `getAllMarkets()`
- `claimTimeRemaining()`
- `calculateAPR()`
- `estimateAPR()`
- `getInterestAccrued()`

---

## [NC-02] isEligible Naming Non-compliance
[Prime: Line 904](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L904)

**Issue Description:** 
The internal function `isEligible` does not adhere to Solidity style guidelines for internal functions, as its name lacks a leading underscore.

**Recommended Mitigation Steps:** 
To align with best practices, rename the function to `_isEligible`.

---
## [NC-03] Incorrect Documentation in Audit Description

**Issue Description:** 
The provided audit sponsor description incorrectly states that "all currently unreleased funds issued can be sent from both `PrimeLiquidityProvider` and `ProtocolShareReserve` to the `Prime` contract by anyone." This statement is inaccurate, as the funds of the PLP are released only when a user claims interest and the Prime token balance is insufficient. Furthermore, the `releaseFunds()` function of PLP can only be called by the Prime token itself.

**Recommended Mitigation Steps:** 
Revise the description/documentation to accurately reflect the process and restrictions associated with the release of funds.

---
## [NC-04] Incorrect Documentation of the Variable totalScoreUpdatesRequired
[Readme.md Line 236](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/README.md?plain=1#L236)

**Issue Description:** 
The documentation inaccurately states that the variable responsible for tracking pending upgrades is named `totalScoreUpdatesRequired`, while the actual variable name is `pendingScoreUpdates`.

**Recommended Mitigation Steps:** 
Update the documentation to use the correct variable name, replacing `totalScoreUpdatesRequired` with `pendingScoreUpdates`.

---
## [NC-05] Indistinct Access Control Definition in Headers
[PrimeLiquidityProvider Line 116](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L116)
[PrimeLiquidityProvider Line 175](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L175)
[PrimeLiquidityProvider Line 214](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L214)

**Issue Description:**
In the `PrimeLiquidityProvider` contract, multiple functions are ambiguously defined with access control annotations, specifically stating either `@custom:access Only Governance` or `@custom:access Only owner`. Consistency in access control definitions is needed.

**Recommended Mitigation Steps:**
Choose a standardized convention for function headers, either `Only Governance` or `Only owner`, and apply it consistently to all affected functions.

---
## [NC-06] Missing Errors in Multiple Function Headers in PrimeLiquidityProvider
[PrimeLiquidityProvider Line 332](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L332)
[PrimeLiquidityProvider Line 344](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L332)

**Issue Description:** 
Several functions in the `PrimeLiquidityProvider` contract throw errors, but these errors are not declared in the function headers. This lack of error declaration can lead to confusion.

**Recommended Mitigation Steps:** 
For each affected function, include the relevant error declarations in the function headers to improve clarity and documentation.
### `_ensureTokenInitialized()`
The function throws the `TokenNotInitialized` error but that is not declared in the header
### Recommended Mitigation Steps:
Add this line to the header:

```solidity
* @custom:error Throw TokenNotInitialized if the address does not own a  token
```

### `_ensureZeroAddress()`
The function throws the `InvalidArguments` error but that is not declared in the header
### Recommended Mitigation Steps:
Add this line to the header:

```solidity
* @custom:error Throw InvalidArguments if the address is 0
```

---
## [NC-07] Missing Errors in Multiple Function Headers in Prime

**Issue Description:**
Several functions in the `Prime` contract throw errors, but these errors are not declared in the function headers. This omission may result in a lack of clarity for developers interacting with the contract.

**Recommended Mitigation Steps:**
To enhance clarity and documentation, include the relevant error declarations in the headers of the affected functions.
### `constructor()`
The function throws the `InvalidAddress` and `InvalidBlocksPerYear` errors but that is not declared in the header.
### Recommended Mitigation Steps:
Add these lines to the header:

```solidity
* @custom:error Throw InvalidAddress if the wbnb address is 0
* @custom:error Throw InvalidAddress if the vbnb address is 0
* @custom:error Throw InvalidBlocksPerYear if the blocks per year are 0
```

### `initialize()`
The function throws the `InvalidAddress` error but that is not declared in the header.
### Recommended Mitigation Steps:
Add these lines to the header:

```solidity
* @custom:error Throw InvalidAddress if the xvsVault address is 0
* @custom:error Throw InvalidAddress if the xvsVaultRewardToken address is 0
* @custom:error Throw InvalidAddress if the protocolShareReserve address is 0
* @custom:error Throw InvalidAddress if the comptroller address is 0
* @custom:error Throw InvalidAddress if the oracle address is 0
* @custom:error Throw InvalidAddress if the primeLiquidityProvider address is 0
```

### `updateScores()`
The function throws the `NoScoreUpdatesRequired` and `UserHasNoPrimeToken` errors but that is not declared in the header.
### Recommended Mitigation Steps:
Add these lines to the header:

```solidity
* @custom:error Throw NoScoreUpdatesRequired if the pendingScoreUpdates is 0
* @custom:error Throw NoScoreUpdatesRequired if the nextScoreUpdateRoundId is 0
* @custom:error Throw UserHasNoPrimeToken if the user doesn't have a prime token
```

### `updateMultipliers()`
The function throws the `MarketNotSupported` error but that is not declared in the header.
### Recommended Mitigation Steps:
Add this line to the header:

```solidity
* @custom:error Throw MarketNotSupported if the market does not exist
```

### `addMarket()`
The function throws the `MarketAlreadyExists` and `InvalidVToken` errors but that is not declared in the header.
### Recommended Mitigation Steps:
Add these lines to the header:

```solidity
* @custom:error Throw MarketAlreadyExists if the market is already in the prime program.
* @custom:error Throw InvalidVToken if the market does not exist. 
```

### `setLimit()`
The function throws the `InvalidLimit` error but that is not declared in the header.
### Recommended Mitigation Steps:
Add this line to the header:

```solidity
* @custom:error Throw InvalidLimit if one of the new total values is smaller than the current value
```

### `claim()`
The function throws the `IneligibleToClaim` and `WaitMoreTime` errors but that is not declared in the header.
### Recommended Mitigation Steps:
Add these lines to the header:

```solidity
* @custom:error Throw IneligibleToClaim if the user has not staked enough XVS
* @custom:error Throw WaitMoreTime if the time since the user has staked is less than the staking period. 
```

### `updateAssetsState()`
The function throws the `InvalidCaller`,  `InvalidComptroller`and `MarketNotSupported` errors but that is not declared in the header.
### Recommended Mitigation Steps:
Add these lines to the header:

```solidity
* @custom:error Throw InvalidCaller if the function is called by someone else than the protocolShareReserve
* @custom:error Throw InvalidComptroller if the address passed as _comptroller parameter is not the comptroller
* @custom:error Throw MarketNotSupported if the vToken address is 0. 
```

### `accrueInterest()`
The function throws the `MarketNotSupported` error but that is not declared in the header.
### Recommended Mitigation Steps:
Add this line to the header:

```solidity
* @custom:error Throw MarketNotSupported if the market is not a prime market
```

### `_mint()`
The function throws the `IneligibleToClaim` and `WaitMoreTime` errors but that is not declared in the header.
### Recommended Mitigation Steps:
Add these lines to the header:

```solidity
* @custom:error Throw IneligibleToClaim if the user does not have a prime token
* @custom:error Throw InvalidLimit if all tokens of that category have already been minted
```

### `_burn()`
The function throws the `UserHasNoPrimeToken` error but that is not declared in the header.
### Recommended Mitigation Steps:
Add this line to the header:

```solidity
* @custom:error Throw UserHasNoPrimeToken if the user has no prime token
```

### `_upgrade()`
The function throws the `InvalidLimit` error but that is not declared in the header.
### Recommended Mitigation Steps:
Add this line to the header:

```solidity
* @custom:error Throw InvalidLimit if all tokens of that category have already been minted
```

### `_checkAlphaArguments()`
The function throws the `InvalidAlphaArguments` error but that is not declared in the header.
### Recommended Mitigation Steps:
Add this line to the header:

```solidity
* @custom:error Throw InvalidAlphaArguments if the denominator is 0 or smaller than the numerator
```

---
## [NC-08] Missing Access Restrictions in Multiple Function Headers in Prime

**Issue Description:** Several functions in the `Prime` contract are access-restricted by the AccessManager, but this access control is not mentioned in their function headers. Properly documenting access control is essential for understanding contract behavior. The affected functions are:
- `updateAlpha()`
- `updateMultipliers()`
- `addMarket()`
- `setLimit()`
- `issue()`
- `burn()`
- `togglePause()`

**Recommended Mitigation Steps:** 
Add the following access control declaration to the headers of all affected functions:
```
* @custom:access Controlled by ACM
```

---
## [NC-09] Bad Indexing Practices
[Prime Line 63](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L63)
[Prime Line 74](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L74)
[Prime Line 82](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L82)

**Issue Description:**
In the events `MintLimitsUpdated`, `AlphaUpdated`, and `MultiplierUpdated`, the new values are indexed incorrectly while the old ones are indexed. Proper indexing is essential for making contract events more informative for users monitoring the contract.

**Recommended Mitigation Steps:** 
Adjust the indexing to focus on the new values rather than the old ones for the mentioned events to provide clearer event data.

---
## [NC-10] Missing Indexing in InterestClaimed
[Prime Line 91](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L91)

**Issue Description:** 
The `InterestClaimed` event lacks indexing for the `amount` parameter, which can hinder users seeking specific event data.

**Recommended Mitigation Steps:** 
Add indexing to the `amount` parameter in the `InterestClaimed` event to improve event data accessibility.

---
## [NC-11] Missing Indexing in mint
[Prime Line 51](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L51)

**Issue Description:**
The `mint` event does not index the `isIrrevocable` parameter, potentially limiting the usefulness of the event data.

**Recommended Mitigation Steps:**
Index the `isIrrevocable` parameter in the `mint` event to enhance the event's informative value.

---
## [NC-12] Functions That Can Be Declared as pure
[Prime Line 809](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L809)
[Prime Line 854](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L854)
[Prime Line 904](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L904)

**Issue Description:** 
The functions `_checkAlphaArguments`, `_xvsBalanceForScore`, and `isEligible` do not read or modify any state variables within the contract. Therefore, they can be declared as `pure` for improved clarity.

**Recommended Mitigation Steps:** 
Declare the functions as `pure` to signify their lack of state-changing behavior and improve contract readability.

---
## [NC-13] claimTimeRemaining() Misleading Name
[Prime Line 478](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L478)

**Issue Description:** 
The function `claimTimeRemaining()` has a misleading name, as it does not return the time remaining for users to claim their tokens but rather the time it takes until users can claim their tokens.

**Recommended Mitigation Steps:** 
Rename the function to `timeUntilClaimRemaining()` to accurately convey its functionality.

---
## [NC-14] Inaccurate Naming of Error MarketAlreadyExists
[Prime Line 28](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L28)

The error `MarketAlreadyExists`is badly named as it is used when checking for if the market is already marked as a prime market, not if the market exists. 


**Issue Description:** 
The error `MarketAlreadyExists` is misleading, as it is used to check if a market is already marked as a prime market, not if the market exists.

**Recommended Mitigation Steps:** 
Rename the error to `MarketIsAlreadyPrimeMarket` to align with its actual usage.

___
## [NC-15] Inconsistent Naming of vToken Address in Function Parameters

**Issue Description:** 
In the contract functions, there is inconsistent naming of the `vToken` address parameter, sometimes referred to as `vToken` and other times as `market`. Consistency in parameter naming enhances code readability.

In the following functions the address is called vToken:
- `addMarket()`
- `claimInterest()`
- `accrueInterest()`
- `getInterestAccrued()`
- `_claimInterest()`
- `_executeBoost()`
- `_interestAccrued()`
- `_getUnderlying()`
- `_incomePerBlock()`
- `_incomeDistributionYearly()`) 
While in these functions the address is called market
- `updateMultipliers()` 
- `accrueInterestAndUpdateScore()`
- `calculateAPR()`
- `_calculateScore()`
- `_updateScore()`

**Recommended Mitigation Steps:** 
Standardize the parameter naming to either `vToken` or `market` across all functions to improve code consistency and clarity.

---
## [NC-16] Invalid Description for lastAccruedBlock
[PrimeLiquidityProvider: Line 23](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L23)

**Issue Description:** The description in the header for the variable `lastAccruedBlock` inaccurately states that it represents the rate at which tokens are distributed to the `Prime` contract. In reality, it signifies the block at which the token was initialized or the last block at which `accrueTokens()` was called.

**Recommended Mitigation Steps:** Revise the description to accurately convey that `lastAccruedBlock` represents the block at which token initialization or accrual occurred.

```
/// @notice The last block at which the tokens interest was accrued (or the token was initialized)
mapping(address => uint256) public lastAccruedBlock;
```
