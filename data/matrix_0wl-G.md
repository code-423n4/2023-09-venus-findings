# Report

## Gas Optimizations

|        | Issue                                                                                                                                              |
| ------ | :------------------------------------------------------------------------------------------------------------------------------------------------- |
| GAS-1  | Internal functions only called once can be inlined to save gas                                                                                     |
| GAS-2  | Functions guaranteed to revert when called by normal users can be marked payable                                                                   |
| GAS-3  | Optimize names to save gas                                                                                                                         |
| GAS-4  | ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR-LOOP AND WHILE-LOOPS |
| GAS-5  | PUBLIC FUNCTIONS TO EXTERNAL                                                                                                                       |
| GAS-6  | Use a more recent version of solidity                                                                                                              |
| GAS-7  | State variables only set in the constructor should be declared immutable                                                                           |
| GAS-8  | Using storage instead of memory for structs/arrays saves gas                                                                                       |
| GAS-9  | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                                                                                |
| GAS-10 | Upgrade to at least 0.8.4                                                                                                                          |

### [GAS-1] Internal functions only called once can be inlined to save gas

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

762:     function _upgrade(address user) internal {

827:     function _updateRoundAfterTokenBurned(address user) internal {

904:     function isEligible(uint256 amount) internal view returns (bool) {

947:     function _incomePerBlock(address vToken) internal view returns (uint256) {

957:     function _distributionPercentage() internal view returns (uint256) {

970:     function _incomeDistributionYearly(address vToken) internal view returns (uint256 amount) {

```

### [GAS-2] Functions guaranteed to revert when called by normal users can be marked payable

#### Description:

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

118:     function initializeTokens(address[] calldata tokens_) external onlyOwner {

177:     function setPrimeToken(address prime_) external onlyOwner {

216:     function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {

```

### [GAS-3] Optimize names to save gas

#### Description:

`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. In this report are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/IPrime.sol

4: interface IPrime {

```

```solidity
File: contracts/Tokens/Prime/Prime.sol

35: contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, MaxLoopsLimitHelper, PrimeStorageV1 {

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

8: contract PrimeLiquidityProvider is AccessControlledV8, PausableUpgradeable {

```

```solidity
File: contracts/Tokens/Prime/PrimeStorage.sol

6: contract PrimeStorageV1 {

```

### [GAS-4] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR-LOOP AND WHILE-LOOPS

#### Description:

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

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

### [GAS-5] PUBLIC FUNCTIONS TO EXTERNAL

#### Description:

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

554:     function accrueInterest(address vToken) public {

597:     function getInterestAccrued(address vToken, address user) public returns (uint256) {

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

249:     function accrueTokens(address token_) public {

276:     function getBlockNumber() public view virtual returns (uint256) {

```

### [GAS-6] Use a more recent version of solidity

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/IPrime.sol

2: pragma solidity ^0.5.16;

```

```solidity
File: contracts/Tokens/Prime/Prime.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/PrimeStorage.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath.sol

4: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

4: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/Scores.sol

3: pragma solidity 0.8.13;

```

### [GAS-7] State variables only set in the constructor should be declared immutable

#### Description:

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD. Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas)

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

103:     constructor(address _wbnb, address _vbnb, uint256 _blocksPerYear) {

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

78:     constructor() {

```

### [GAS-8] Using storage instead of memory for structs/arrays saves gas

#### Description:

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

174:     function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {

176:         PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

200:     function updateScores(address[] memory users) external {

469:     function getAllMarkets() external view returns (address[] memory) {

683:             address[] memory assets = new address[](1);

```

### [GAS-9] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.
Note that this optimization seems to be dependent on usage of a more recent Solidity version.

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

370:        if (tokens[user].isIrrevocable) {
                _accrueInterestAndUpdateScore(user);
            } else {
                _burn(user);
            }

421:    if (paused()) {
            _unpause();
        } else {
            _pause();
        }

482:    if (totalTimeStaked < STAKING_PERIOD) {
            return STAKING_PERIOD - totalTimeStaked;
        } else {
            return 0;
        }

710:    if (isIrrevocable) {
            totalIrrevocable++;
        } else {
            totalRevocable++;
        }

744:    if (tokens[user].isIrrevocable) {
            totalIrrevocable--;
        } else {
            totalRevocable--;
        }

857:    if (xvs > MAXIMUM_XVS_CAP) {
            return MAXIMUM_XVS_CAP;
        } else {
            return xvs;
        }

931:    if (vToken == VBNB) {
            return WBNB;
        } else {
            return IVToken(vToken).underlying();
        }

```

### [GAS-10] Upgrade to at least 0.8.4

#### Description:

Use pragma solidity bigger than 0.8.0 - Using newer compiler versions and the optimizer gives gas optimizations and additional safety checks for free!
Safemath by default from 0.8.0 (can be more gas efficient than some library-based safemath.)
OVERFLOW/UNDERFLOW - An overflow/underflow happens when an arithmetic operation reaches the maximum or minimum size of a type. For instance if a number is stored in the uint8 type, it means that the number is stored in a 8 bits unsigned number ranging from 0 to 2^8-1. In computer programming, an integer overflow occurs when an arithmetic operation attempts to create a numeric value that is outside of the range that can be represented with a given number of bits – either larger than the maximum or lower than the minimum representable value.
https://swcregistry.io/docs/SWC-101
[Low-level inliner](https://blog.soliditylang.org/2021/03/02/saving-gas-with-simple-inliner/) from 0.8.2, leads to cheaper runtime gas. Especially relevant when the contract has small functions. For example, OpenZeppelin libraries typically have a lot of small helper functions and if they are not inlined, they cost an additional 20 to 40 gas because of 2 extra jump instructions and additional stack operations needed for function calls.
[Optimizer improvements in packed structs](https://blog.soliditylang.org/2021/03/23/solidity-0.8.3-release-announcement/#optimizer-improvements): Before 0.8.3, storing packed structs, in some cases, used additional storage read operation. After [EIP-2929](https://eips.ethereum.org/EIPS/eip-2929), if the slot was already cold, this means unnecessary stack operations and extra deploy time costs. However, if the slot was already warm, this means an additional cost of 100 gas alongside the same unnecessary stack operations and extra deploy time costs.)
[Custom errors](https://blog.soliditylang.org/2021/04/21/custom-errors) 24 from 0.8.4, leads to cheaper deploy time cost and run time cost. Note: the run time cost is only relevant when the revert condition is met. In short, replace revert strings with custom errors.
[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/IPrime.sol

2: pragma solidity ^0.5.16;

```

#### Recommended Mitigation Steps:

Upgrade to at least 0.8.4
