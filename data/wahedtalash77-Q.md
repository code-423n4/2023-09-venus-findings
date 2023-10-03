|     |     |     |
| --- | --- | --- |
| ID  | Description | Severity |
| \[L-01\] | Insufficient coverage | Low |
| \[N-02\] | Showing the actual values of numbers in NatSpec comments makes checking and reading code easier | Non-Critical |
| \[N-03\] | Inconsistent Solidity Versions | Non-Critical |
| \[N-04\] | Use a single file for all system-wide constants | Non-Critical |
| \[N‑05\] | Contracts should have full test coverage | Non-Critical |
| \[N-06\] | Solidity version | Non-Critical |
| \[N-07\] | NatSpec comments should be increased in contracts | Non-Critical |
| \[N-08\] | Function writing that does not comply with the Solidity Style Guide | Non-Critical |
| \[N-09\] | Use SMTChecker | Non-Critical |
| \[N-10\] | Assembly Codes Specific – Should Have Comments | Non-Critical |
| \[N-11\] | Use the delete keyword instead of assigning a value of 0 | Non-Critical |
| \[N-12\] | Function writing that does not comply with the Solidity Style Guide | Non-Critical |
| \[S-01\] | You can explain the operation of critical functions in NatSpec with an infographic. | Suggestion |

## \[L-01\] Insufficient coverage

Description
The test coverage rate of the project is ~96%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

## \[N-01\] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

```
uint256 public constant MAX_DISTRIBUTION_SPEED = 1e18; // 1_000_000_000_000_000_000

    /// @notice exp scale
    uint256 internal constant EXP_SCALE = 1e18; // 1_000_000_000_000_000_000
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L12C4-L15C48

```
int256 internal constant EXP_SCALE = 1e18; // 1_000_000_000_000_000_000
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L31

## \[N-02 Inconsistent Solidity Versions

```Solidity
File: contracts/Tokens/Prime/Prime.sol
2: pragma solidity 0.8.13;

// All contracts use 0.8.13 expect Iprime.sol

contracts/Tokens/Prime/IPrime.sol
2: pragma solidity ^0.5.16;
```

**Recommendation**
Versions must be consistent with each other.

## \[N-03\] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

## \[N‑04\] Contracts should have full test coverage

While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

## \[N-05\] Solidity version

All contracts are using 0.8.13. Consider updating to the latest version 0.8.19 to ensure the compiler contains the latest security fixes.

## \[N-06\] NatSpec comments should be increased in contracts

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## \[N-07\] Function writing that does not comply with the Solidity Style Guide

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## \[N-08\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-09\] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of
Solidity, which can make the code more insecure and more error-prone.

## \[N-10\] Use the delete keyword instead of assigning a value of 0

Using the ‘delete’ keyword instead of assigning a ‘0’ value is a detailed optimization that increases code readability and audit quality, and clearly indicates the intent.

Other hand, if use delete instead 0 value assign , it will be gas saved.

## \[N-11\] Function writing that does not comply with the Solidity Style Guide

Context
All Contracts

Description
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

## \[S-01\] You can explain the operation of critical functions in NatSpec with an infographic.