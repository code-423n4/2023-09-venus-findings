### Gas Optimizations

Total **17 instances** over **2 issues** with **8315** gas saved:

|ID|Issue|Instances|Gas saved|
|:--:|:---|:--:|:--:|
| [G-1](#g-01-state-variables-can-be-packed-to-use-fewer-storage-slots) | State variables can be packed to use fewer storage slots | 2 | 8000 |
| [G-2](#g-02-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable) | Functions guaranteed to revert when called by normal users can be marked payable | 15 | 315 |

### [G-01] State variables can be packed to use fewer storage slots

Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions, we will effectively save ~2000 gas with every subsequent `SLOAD` for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`

We can pack together `totalIrrevocable`, `totalRevocable`, `revocableLimit`, and `irrevocableLimit` into a single storage slot by making them all `uint64`. The reason we can do this is because `uint64` is a very big number (`type(uint64).max = 18,446,744,073,709,551.615`) and these variables will never be bigger than that number. We save 3 SLOTs (~6000 gas). This would require changing some events and functions that use these variables.

Code Snippet: [PrimeStorage.sol#L48-L58](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L48-L58)

```diff
/// @notice  Tracks total irrevocable tokens minted
- uint256 public totalIrrevocable;
+ uint64 public totalIrrevocable;

/// @notice  Tracks total revocable tokens minted
- uint256 public totalRevocable;
+ uint64 public totalRevocable;

/// @notice  Indicates maximum revocable tokens that can be minted
- uint256 public revocableLimit;
+ uint64 public revocableLimit;

/// @notice  Indicates maximum irrevocable tokens that can be minted
- uint256 public irrevocableLimit;
+ uint64 public irrevocableLimit;
```

We can pack together `totalScoreUpdatesRequired` and `pendingScoreUpdates` into a single storage slot by making them `uint128`. The reason is the same as above, they will never exceed `type(uint128).max = 340,282,366,920,938,463,463,374,607,431,768,211,455`. We save 1 SLOT (~2000 gas). This would require changing some events and functions that use these variables.

Code Snippet: [PrimeStorage.sol#L93-L97](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L93-L97)

```diff
/// @notice total number of accounts whose score needs to be updated
- uint256 public totalScoreUpdatesRequired;
+ uint128 public totalScoreUpdatesRequired;

/// @notice total number of accounts whose score is yet to be updated
- uint256 public pendingScoreUpdates;
+ uint128 public pendingScoreUpdates;
```

### [G-02] Functions guaranteed to revert when called by normal users can be marked payable

If access control is used where the function will revert if a normal user tries to pay the function, marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a function is payable or not. Saving ~21 gas per function.

Code Snippet in Prime.sol

- [updateAlpha](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L237)
- [updateMultipliers](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L263)
- [addMarket](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L288)
- [setLimit](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L316)
- [issue](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331)
- [burn](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L411)
- [togglePause](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L419)
- [updateAssetsState](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L452)

Code Snippet in PrimeLiquidityProvider.sol

- [initializeTokens](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118)
- [pauseFundsTransfer](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L132)
- [resumeFundsTransfer](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L141)
- [setTokensDistributionSpeed](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L153)
- [setPrimeToken](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177)
- [releaseFunds](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192)
- [sweepToken](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216)