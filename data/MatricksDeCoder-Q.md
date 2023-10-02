### LOW-1 Arrays lack checks for address(0) values 

1. https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L331

2. https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L9

3. https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118 

4. https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L153 

The above has examples of addresses used in arrays e.g users, tokens such that each individual address e.g users[i] or tokens[i] is not checked for address(0) before using it in function logic. 

The above can lead to unexpected errors, mints to address(0), updates to address(0) thinking a particular address has been updated when not, unexpected behavior or even potentially introduce bugs 

### NC-1 - Interfaces should be placed in their own folder 

Interface file IPrime.sol should be placed in interfaces folder 
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/IPrime.sol

**Impact**-> It affects the code  maintainability, readability, auditability and code organization

**Recommendation** -> It is recommended to move all interfaces like IPrime.sol into the /contracts/Tokens/Prime/Interfaces folder 

### NC-2 - Contracts using different licenses 

Some contracts in scope are using 
// SPDX-License-Identifier: BSD-3-Clause licence  e.g Prime.sol, IPrime.sol, PrimeStorage.sol 
whereas some are using 
// SPDX-License-Identifier: MIT e.g Scores.sol, FixedMath.sol, FixedMathOx.sol 

**Impact**-> It may have an impact on the legal, open source, reputation, reusability, aspects of the code etc

**Recommendation** -> It is recommended contracts use similar license e.g 
// SPDX-License-Identifier: BSD-3-Clause

### NC-3 - Imports should be organized more systematically 

Interfaces are not imported first in the contracts e.g Prime.sol
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L13

**Impact**-> It affects the code  organization, maintainability, readability, auditability 

**Recommendation** -> Interfaces should be imported first, followed by each of the interfaces they use, then followed by all other files. Check all contract files and apply this recommended best practise 

### NC-4 - Lack sanity checks function inputs values that must != 0  

1. https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L141 
uint256 _loopsLimit in used in MaxLoopsLimitHelper must never be 0 

2. https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L263C56-L263C98
supplyMultiplier, uint256 borrowMultiplier must never be 0 as they are critical multipliers 

3. https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L288
supplyMultiplier, uint256 borrowMultiplier when adding market must never be 0 as they are critical multipliers

4. /contracts/Tokens/Prime/libs/FixedMath.sol critical library functions with some values that must no be zero but is not checked
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/libs/FixedMath.sol#L22
```
 function toFixed(uint256 n, uint256 d) internal pure returns (int256) {
        if (d.toInt256() < n.toInt256()) revert InvalidFraction(n, d);

        return (n.toInt256() * FixedMath0x.FIXED_1) / int256(d.toInt256());
    }
```
input 'd' is not checked if !=0 which leads to division by 0 and may causes unexpected reverts 

```
 function uintDiv(uint256 u, int256 f) internal pure returns (uint256) {
        if (f < 0) revert InvalidFixedPoint();
        // multiply `u` by FIXED_1 to cancel out the built-in FIXED_1 in f
        return uint256((u.toInt256() * FixedMath0x.FIXED_1) / f);
    }
```
check for f should be f <=0 to avoid 0 value f so that there is no division by zero in above

**Impact**-> It can lead to errors, code not behaving as expected, unexpected reverts, potential bugs etc 

**Recommendation** -> It is recommended to ensure all cases where inputs of values = 0 in functions are checked and ensure there is revert in such cases so that zero values are not allowed e.g
```
require(inputValue != 0, "error string")
```

### NC-5 - Inconsistent underscore in function arguments/parameters 

In some functions the parameters/arguments make use of underscore e.g 
```
// contracts/Tokens/Prime/Prime.sol
function updateAlpha(uint128 _alphaNumerator, uint128 _alphaDenominator) external 
```
Whereas other functions do not make use of this best practice e.g 
```
// contracts/Tokens/Prime/Prime.sol
function updateMultipliers(address market, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
```

Whereas we see that in PrimeLiquidityProvider.sol 
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol
the format of address token_ trailing underscore is used 

**Impact**-> It affects the code  maintainability, readability, auditability, and inconsistency

**Recommendation** -> It is recommended to be consistent in implementation of underscore to functions for the parameters/arguments e.g make use of _underscore all functions as done in the initializer Prime.sol below 
```
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
```

### NC6 - Custom Errors should be prefixed by contract name

1. https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L20
Pime.sol lines 20-32 e.g
```
error InvalidAddress();
error InvalidBlocksPerYear();
error InvalidAlphaArguments();
error InvalidVToken();
```

2. https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L57
PrimeLiquidityProvider.sol lines 57 - 75 e.g
```
 /// @notice Error thrown when funds transfer is paused
    error FundsTransferIsPaused();

    /// @notice Error thrown when intrest accrue is called for not initialized token
    error TokenNotInitialized(address token_);
``

**Impact**-> Not naming errors with indication of contract can make error tracking, debugging, analysis difficult as error data bubbles through chain of calls so it may not be clear if error contract receives is from where

**Recommendation** -> It is recommended the errors be named in accordance to the name of the contract in which they will be thrown e. g error
```javascript
error Prime__InvalidAddress();
error Prime__InvalidBlocksPerYear();
error Prime__InvalidAlphaArguments();
error Prime__InvalidVToken();
```

### NC-7 - Consider using ternary statement for simple if/else cases 

There are some missed opportunities to use shorthand way of if/else using the ternary operator, as it increases readability and reduces the number of lines of code.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L419 
```
 /**
     * @notice To pause or unpause claiming of interest
     */
    function togglePause() external {
        _checkAccessAllowed("togglePause()");
        if (paused()) {
            _unpause();
        } else {
            _pause();
        }
    }
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L854C1-L860C6 
```
    function _xvsBalanceForScore(uint256 xvs) internal view returns (uint256) {
        if (xvs > MAXIMUM_XVS_CAP) {
            return MAXIMUM_XVS_CAP;
        } else {
            return xvs;
        }
    }
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L482C4-L482C4
```
    function claimTimeRemaining(address user) external view returns (uint256) {
        if (stakedAt[user] == 0) return STAKING_PERIOD;

        uint256 totalTimeStaked = block.timestamp - stakedAt[user];
        if (totalTimeStaked < STAKING_PERIOD) {
            return STAKING_PERIOD - totalTimeStaked;
        } else {
            return 0;
        }
    }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L710
```
function _mint(bool isIrrevocable, address user) internal {
        if (tokens[user].exists) revert IneligibleToClaim();

        tokens[user].exists = true;
        tokens[user].isIrrevocable = isIrrevocable;

        if (isIrrevocable) {
            totalIrrevocable++;
        } else {
            totalRevocable++;
        }

        if (totalIrrevocable > irrevocableLimit || totalRevocable > revocableLimit) revert InvalidLimit();

        emit Mint(user, isIrrevocable);
    }

```
More examples include 
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L746 
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L933


Impact-> It affects the code maintainability, readability, auditability and code organization

Recommendation -> It is recommended to apply ternary in relevant cases as in examples given above

