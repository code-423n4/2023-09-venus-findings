## QA
---

### natSpec missing [1]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol

```solidity
// @param and @return missing
53:    function ln(int256 x) internal pure returns (int256 r) {
58:    function exp(int256 x) internal pure returns (int256 r) {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/Scores.sol

```solidity
10:  library Scores {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

```solidity
8:  contract PrimeLiquidityProvider is AccessControlledV8, PausableUpgradeable {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol

```solidity
6:  contract PrimeStorageV1 {
7:    struct Token {
12:    struct Market {
20:    struct Interest {
26:    struct PendingInterest {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/IPrime.sol

```solidity
4:  interface IPrime {
5:    function xvsUpdated(address user) external;
7:    function accrueInterestAndUpdateScore(address user, address market) external;
9:    function accrueInterest(address vToken) external;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

```solidity
// @param missing
827:    function _updateRoundAfterTokenBurned(address user) internal {
```

---

### State variable and function names [2]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol

```solidity
// private and internal `functions` should preppend with `underline`
51:    function ln(int256 x) internal pure returns (int256 r) {
140:    function exp(int256 x) internal pure returns (int256 r) {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol

```solidity
// private and internal `functions` should preppend with `underline`
22:    function toFixed(uint256 n, uint256 d) internal pure returns (int256) {
34:    function uintDiv(uint256 u, int256 f) internal pure returns (uint256) {
46:    function uintMul(uint256 u, int256 f) internal pure returns (uint256) {
53:    function ln(int256 x) internal pure returns (int256 r) {
58:    function exp(int256 x) internal pure returns (int256 r) {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/Scores.sol

```solidity
// private and internal `functions` should preppend with `underline`
22:    function calculateScore(
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol

```solidity
// private and internal `functions` should preppend with `underline`
70:    address[] internal allMarkets;
79:    address internal xvsVault;
82:    address internal xvsVaultRewardToken;
85:    uint256 internal xvsVaultPoolId;
```

---

### Version [3]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/IPrime.sol

```solidity
// use a more up to date version.
2:  pragma solidity ^0.5.16;
```

