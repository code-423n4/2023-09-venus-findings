# VENUS PRIME GAS OPTIMIZATIONS

## INTRODUCTION 
Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."

Emphasizing the significance of our recommendations, we strive to improve the code's efficiency while maintaining its readability. We recognize the importance of code that is easily maintainable and understandable for both developers and auditors.

## TABLE OF FINDINGS

| Number |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| G-01| Avoid using state variable in emit | 1 | 97 |
| G-02| Refactor internal functions to avoid unnecessary SLOAD | 1 | 97 |
| G-03| Call `block.number` direclty rather than use function call | 2 | 40 |
| G-04| Use v4.9.0 or above OpenZeppelin contracts | 1 | - |
| G-05| Revert() statements that check input arguments should be at the top of the function | 1 |  - |
| G-06| Sort Solidity operations using short-circuit mode | 3 | - |
| G-07| Declaring Unnecessary variables. | 2 |  6 |
| G-08| Use += for mappings | 3 | 120 |




## [G-01] Avoid using state variable in emit
In scenarios where you have a memory, calldata variable or parameter with the same value as the state variable you should use the memory, calldata variable or parameter in the emit statement rather than state variable.

### 1 Instance
1. #### Refactor `emit UpdatedAssetsState(comptroller, asset)` to use `_comptroller` in place of `comptroller` to avoid reading from state.
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L462
```sollidity
file: contracts/Tokens/Prime/Prime.sol

452:    function updateAssetsState(address _comptroller, address asset) external {
453:        if (msg.sender != protocolShareReserve) revert InvalidCaller();
454:        if (comptroller != _comptroller) revert InvalidComptroller();
455:
456:        address vToken = vTokenForAsset[asset];
457:        if (vToken == address(0)) revert MarketNotSupported();
458:
459:        IVToken market = IVToken(vToken);
460:        unreleasedPSRIncome[_getUnderlying(address(market))] = 0;
461:
462:        emit UpdatedAssetsState(comptroller, asset);    //@audit use _comptroller to avoid reading from state.
463:    }
```
We can avoid 1 `SLOAD(Gwarmaccess)` if we use `_comptroller` in place of `comptroller` in the `emit UpdatedAssetsState(comptroller, asset)` since the `if (comptroller != _comptroller) revert InvalidComptroller()` ensures that both `comptroller` and `_comptroller` are equal.
```diff
diff --git a/contracts/Tokens/Prime/Prime.sol b/contracts/Tokens/Prime/Prime.sol
index 2be244f..97af5c1 100644
--- a/contracts/Tokens/Prime/Prime.sol
+++ b/contracts/Tokens/Prime/Prime.sol
@@ -459,7 +459,7 @@ contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, M
         IVToken market = IVToken(vToken);
         unreleasedPSRIncome[_getUnderlying(address(market))] = 0;

-        emit UpdatedAssetsState(comptroller, asset);
+        emit UpdatedAssetsState(_comptroller, asset);
     }
```
```
Estimated gas saved: 97 gas units
```



## [G-02] Refactor internal functions to avoid unnecessary SLOAD
The functions below read storage slots that are previously read in the functions that invoke them. We can refactor the /internal functions to pass cached storage variables as stack variables and avoid the extra storage reads that would otherwise take place in the internal functions.

### 1 Instance
1. #### Refactor `_distributionPercentage()` and `_incomeDistributionYearly()` to avoid 1`SLOAD`
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L957-#L979
```solidity
file: contracts/Tokens/Prime/Prime.sol

957:    function _distributionPercentage() internal view returns (uint256) {
958:        return
959:            IProtocolShareReserve(protocolShareReserve).getPercentageDistribution(
960:                address(this),
961:                IProtocolShareReserve.Schema.SPREAD_PRIME_CORE
962:            );
963:    }
.
.
.
970:    function _incomeDistributionYearly(address vToken) internal view returns (uint256 amount) {
971:        uint256 totalIncomePerBlockFromMarket = _incomePerBlock(vToken);
972:        uint256 incomePerBlockForDistributionFromMarket = (totalIncomePerBlockFromMarket * _distributionPercentage()) /
973:            IProtocolShareReserve(protocolShareReserve).MAX_PERCENT();
974:        amount = BLOCKS_PER_YEAR * incomePerBlockForDistributionFromMarket;
975:
976:        uint256 totalIncomePerBlockFromPLP = IPrimeLiquidityProvider(primeLiquidityProvider)
977:            .getEffectiveDistributionSpeed(_getUnderlying(vToken));
978:        amount += BLOCKS_PER_YEAR * totalIncomePerBlockFromPLP;
979:    }
```
We could avoid reading the value of `protocolShareReserve` from state in the `_distributionPercentage()` function if we refactore the code as shown below:
```diff
diff --git a/contracts/Tokens/Prime/Prime.sol b/contracts/Tokens/Prime/Prime.sol
index 2be244f..a53a725 100644
--- a/contracts/Tokens/Prime/Prime.sol
+++ b/contracts/Tokens/Prime/Prime.sol
@@ -954,9 +954,9 @@ contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, M
      * @notice the percentage of income we distribute among the prime token holders
      * @return percentage the percentage returned without mantissa
      */
-    function _distributionPercentage() internal view returns (uint256) {
+    function _distributionPercentage(address _protocolShareReserve) internal view returns (uint256) {
         return
-            IProtocolShareReserve(protocolShareReserve).getPercentageDistribution(
+            IProtocolShareReserve(_protocolShareReserve).getPercentageDistribution(
                 address(this),
                 IProtocolShareReserve.Schema.SPREAD_PRIME_CORE
             );
@@ -969,8 +969,9 @@ contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, M
      */
     function _incomeDistributionYearly(address vToken) internal view returns (uint256 amount) {
         uint256 totalIncomePerBlockFromMarket = _incomePerBlock(vToken);
-        uint256 incomePerBlockForDistributionFromMarket = (totalIncomePerBlockFromMarket * _distributionPercentage()) /
-            IProtocolShareReserve(protocolShareReserve).MAX_PERCENT();
+        address _protocolShareReserve = protocolShareReserve;
+        uint256 incomePerBlockForDistributionFromMarket = (totalIncomePerBlockFromMarket * _distributionPercentage(_protocolShareReserve)) /
+            IProtocolShareReserve(_protocolShareReserve).MAX_PERCENT();
         amount = BLOCKS_PER_YEAR * incomePerBlockForDistributionFromMarket;

         uint256 totalIncomePerBlockFromPLP = IPrimeLiquidityProvider(primeLiquidityProvider)
```
```
Estimated gas saved: 97 gas units.
```



## [G-03] Call `block.number` direclty rather than use function call
`block.number` is a global variable so rather than writing a function to return `block.number` and calling the function whenever you need access `block.number` you should call `block.number` directly these saves deployment gas and having to do two JUMP instructions, along with stack setup when calling the `getBlockNumber()` function.

###  2 Instances
1. #### Use `block.number` directly
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L254
```solidity
file: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

249:    function accrueTokens(address token_) public {
250:        _ensureZeroAddress(token_);
251:
252:        _ensureTokenInitialized(token_);
253:
254:        uint256 blockNumber = getBlockNumber(); //@audit use block.number directly
255:        uint256 deltaBlocks = blockNumber - lastAccruedBlock[token_];
256:
257:        if (deltaBlocks > 0) {
258:            uint256 distributionSpeed = tokenDistributionSpeeds[token_];
259:            uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
260:
261:            uint256 balanceDiff = balance - tokenAmountAccrued[token_];
262:            if (distributionSpeed > 0 && balanceDiff > 0) {
263:                uint256 accruedSinceUpdate = deltaBlocks * distributionSpeed;
264:                uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);
265:
266:                tokenAmountAccrued[token_] += tokenAccrued;
267:                emit TokensAccrued(token_, tokenAccrued);
268:            }
269:
270:            lastAccruedBlock[token_] = blockNumber;
271:        }
272:    }
```
We could save up to `20` gas units in the `accrueTokens()` function above if we use `block.number` directly rather using call the `getBlockNumber()` function and caching the value in variable `blockNumber`. The code should be refactored as shown in the diff below:
```diff
diff --git a/contracts/Tokens/Prime/PrimeLiquidityProvider.sol b/contracts/Tokens/Prime/PrimeLiquidityProvider.sol
index 0cbb17a..b4a8b7a 100644
--- a/contracts/Tokens/Prime/PrimeLiquidityProvider.sol
+++ b/contracts/Tokens/Prime/PrimeLiquidityProvider.sol
@@ -251,8 +251,7 @@ contract PrimeLiquidityProvider is AccessControlledV8, PausableUpgradeable {

         _ensureTokenInitialized(token_);

-        uint256 blockNumber = getBlockNumber();
-        uint256 deltaBlocks = blockNumber - lastAccruedBlock[token_];
+        uint256 deltaBlocks = block.number - lastAccruedBlock[token_];

         if (deltaBlocks > 0) {
             uint256 distributionSpeed = tokenDistributionSpeeds[token_];
@@ -267,7 +266,7 @@ contract PrimeLiquidityProvider is AccessControlledV8, PausableUpgradeable {
                 emit TokensAccrued(token_, tokenAccrued);
             }

-            lastAccruedBlock[token_] = blockNumber;
+            lastAccruedBlock[token_] = block.number;
         }
     }
```
```
Estimated gas saved: 20 gas units
```

2. #### Use `block.number` directly
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L288
```solidity
file: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

286:    function _initializeToken(address token_) internal {
287:        _ensureZeroAddress(token_);
288:        uint256 blockNumber = getBlockNumber();  //@audit use block.number directly
289:        uint256 initializedBlock = lastAccruedBlock[token_];
290:
291:        if (initializedBlock > 0) {
292:            revert TokenAlreadyInitialized(token_);
293:        }
294:
295:        /*
296:         * Update token state block number
297:         */
298:        lastAccruedBlock[token_] = blockNumber;
299:
300:        emit TokenDistributionInitialized(token_);
301:    }
```
We could save up to `20` gas units in the `_initializeToken()` function above if we use `block.number` directly rather using call the `getBlockNumber()` function and caching the value in variable `blockNumber`. The code should be refactored as shown in the diff below:
```diff
diff --git a/contracts/Tokens/Prime/PrimeLiquidityProvider.sol b/contracts/Tokens/Prime/PrimeLiquidityProvider.sol
index 0cbb17a..ce6db0d 100644
--- a/contracts/Tokens/Prime/PrimeLiquidityProvider.sol
+++ b/contracts/Tokens/Prime/PrimeLiquidityProvider.sol
@@ -285,7 +285,7 @@ contract PrimeLiquidityProvider is AccessControlledV8, PausableUpgradeable {
      */
     function _initializeToken(address token_) internal {
         _ensureZeroAddress(token_);
-        uint256 blockNumber = getBlockNumber();
+
         uint256 initializedBlock = lastAccruedBlock[token_];

         if (initializedBlock > 0) {
@@ -295,7 +295,7 @@ contract PrimeLiquidityProvider is AccessControlledV8, PausableUpgradeable {
         /*
          * Update token state block number
          */
-        lastAccruedBlock[token_] = blockNumber;
+        lastAccruedBlock[token_] = block.number;

         emit TokenDistributionInitialized(token_);
     }
```
```
Estimated gas saved: 20 gas units
```



## [G-04] Use v4.9.0 or above OpenZeppelin contracts
From your [package.json file](https://github.com/code-423n4/2023-09-venus/blob/main/package.json#L34) it shows that OpenZeppelin v4.8.3 was used i would recommend you use v4.9.0 or above as it provides many gas optimizations that would help make the protocol gas efficient. https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0




## [G-05] Revert() statements that check input arguments should be at the top of the function
Checks that involve input arguments should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### 1 Instance
1. ### Move `if (comptroller != _comptroller) revert InvalidComptroller()` check to top of function
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L453
```solidity
file: contracts/Tokens/Prime/Prime.sol

425:    function updateAssetsState(address _comptroller, address asset) external {
426:        if (msg.sender != protocolShareReserve) revert InvalidCaller(); 
427:        if (comptroller != _comptroller) revert InvalidComptroller();   //@audit move check to top of function
428:
429:        address vToken = vTokenForAsset[asset];
430:        if (vToken == address(0)) revert MarketNotSupported();
440:
441:        IVToken market = IVToken(vToken);
442:        unreleasedPSRIncome[_getUnderlying(address(market))] = 0;
443:
444:        emit UpdatedAssetsState(comptroller, asset);
445:    }
```
Since the `if (comptroller != _comptroller) revert InvalidComptroller()` performs a check on the `_comptroller` parameter it would be better if we perform a check on it first so that if the value of `_comptroller` parameter is wrong the function could revert without performing gas consuming opertions of `if (msg.sender != protocolShareReserve) revert InvalidCaller()` which involves reading from state. The code could be refactored as shown below:
```diff
diff --git a/contracts/Tokens/Prime/Prime.sol b/contracts/Tokens/Prime/Prime.sol
index 2be244f..0a6770e 100644
--- a/contracts/Tokens/Prime/Prime.sol
+++ b/contracts/Tokens/Prime/Prime.sol
@@ -450,9 +450,9 @@ contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, M
      * @param asset The address of the asset whose income is distributed
      */
     function updateAssetsState(address _comptroller, address asset) external {
-        if (msg.sender != protocolShareReserve) revert InvalidCaller();
         if (comptroller != _comptroller) revert InvalidComptroller();
-
+        if (msg.sender != protocolShareReserve) revert InvalidCaller();
+
         address vToken = vTokenForAsset[asset];
         if (vToken == address(0)) revert MarketNotSupported();
```




## [G-06] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

f(x) is a low gas cost operation \
g(y) is a high gas cost operation 

Sort operations with different gas costs as follows \
f(x) || g(y) \
f(x) && g(y)

### 3 Instances
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L369
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L377
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L379
```solidity
file: contracts/Tokens/Prime/Prime.sol

365:    function xvsUpdated(address user) external {
366:        uint256 totalStaked = _xvsBalanceOfUser(user);
367:        bool isAccountEligible = isEligible(totalStaked);
368:
369:        if (tokens[user].exists && !isAccountEligible) {    //@audit apply short circuiting rule
370:            if (tokens[user].isIrrevocable) {
371:                _accrueInterestAndUpdateScore(user);
372:            } else {
373:                _burn(user);
374:            }
375:        } else if (!isAccountEligible && !tokens[user].exists && stakedAt[user] > 0) {
376:            stakedAt[user] = 0;
377:        } else if (stakedAt[user] == 0 && isAccountEligible && !tokens[user].exists) {  //@audit apply short circuiting rule
378:            stakedAt[user] = block.timestamp;
379:        } else if (tokens[user].exists && isAccountEligible) {   //@audit apply short-circuiting rule
380:            _accrueInterestAndUpdateScore(user);
381:        }
382:    }
```
From the `xvsUpdated()` function above we can see that `isAccountEligible` involves just stack read operations and is less gass consuming than `tokens[user].exists` and `stakedAt[user] > 0` which involves state reading operations therefore we could apply short-circuiting rules such that operations of `isAccountEligible` comes first in the conditional and the `isAccountEligible` operations result to false the EVM would not need to perform gas consuming opertions of `tokens[user].exists` and `stakedAt[user] > 0`. The code should be refactored as shown below: 
```diff
diff --git a/contracts/Tokens/Prime/Prime.sol b/contracts/Tokens/Prime/Prime.sol                     
index 2be244f..69836fb 100644                                                                        
--- a/contracts/Tokens/Prime/Prime.sol                                                               
+++ b/contracts/Tokens/Prime/Prime.sol                                                               
@@ -366,7 +366,7 @@ contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, M 
         uint256 totalStaked = _xvsBalanceOfUser(user);                                              
         bool isAccountEligible = isEligible(totalStaked);                                           
                                                                                                     
-        if (tokens[user].exists && !isAccountEligible) {                                            
+        if (!isAccountEligible &&  tokens[user].exists) {                                           
             if (tokens[user].isIrrevocable) {                                                       
                 _accrueInterestAndUpdateScore(user);                                                
             } else {                                                                                
@@ -374,9 +374,9 @@ contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, M 
             }                                                                                       
         } else if (!isAccountEligible && !tokens[user].exists && stakedAt[user] > 0) {              
             stakedAt[user] = 0;                                                                     
-        } else if (stakedAt[user] == 0 && isAccountEligible && !tokens[user].exists) {              
+        } else if (isAccountEligible && stakedAt[user] == 0 && !tokens[user].exists) {              
             stakedAt[user] = block.timestamp;                                                       
-        } else if (tokens[user].exists && isAccountEligible) {                                      
+        } else if (isAccountEligible && tokens[user].exists) {                                      
             _accrueInterestAndUpdateScore(user);                                                    
         }                                                                                           
     }                                                                                               
```




## [G-07] Declaring Unnecessary variables
some varibles were defined even though they are used once. Not defining variables can reduce gas cost and contract size.

### 2 Instances
1. #### Declaration of `totalStaked` variable is unnecessary
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L366
```solidity
file: contracts/Tokens/Prime/Prime.sol

365:    function xvsUpdated(address user) external {
366:        uint256 totalStaked = _xvsBalanceOfUser(user);  //@audit totalStaked variable unnecessary
367:        bool isAccountEligible = isEligible(totalStaked);
368:
369:        if (tokens[user].exists && !isAccountEligible) {
370:            if (tokens[user].isIrrevocable) {
371:                _accrueInterestAndUpdateScore(user);
372:            } else {
373:                _burn(user);
374:            }
375:        } else if (!isAccountEligible && !tokens[user].exists && stakedAt[user] > 0) {
376:            stakedAt[user] = 0;
377:        } else if (stakedAt[user] == 0 && isAccountEligible && !tokens[user].exists) {
378:            stakedAt[user] = block.timestamp;
379:        } else if (tokens[user].exists && isAccountEligible) {
380:            _accrueInterestAndUpdateScore(user);
381:        }
382:    }
```
In the `xvsUpdated()` function above a variable `totalStaked` was declared but was only used once in the function. We could use the `_xvsBalanceOfUser(user)` function directly in place of the `totalStaked` variable. The diff below shows how the code should be refactored: 
```diff
diff --git a/contracts/Tokens/Prime/Prime.sol b/contracts/Tokens/Prime/Prime.sol
index 2be244f..801d4a2 100644
--- a/contracts/Tokens/Prime/Prime.sol
+++ b/contracts/Tokens/Prime/Prime.sol
@@ -363,8 +363,8 @@ contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, M
      * @param user the account address whose balance was updated
      */
     function xvsUpdated(address user) external {
-        uint256 totalStaked = _xvsBalanceOfUser(user);
-        bool isAccountEligible = isEligible(totalStaked);
+
+        bool isAccountEligible = isEligible(_xvsBalanceOfUser(user));

         if (tokens[user].exists && !isAccountEligible) {
             if (tokens[user].isIrrevocable) {
```
```
Estimated gas saved: 3 gas units.
```

2. #### Declaration of `lastBlockAccrued` variable is unnecessary
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L333
```solidity
file: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

332:    function _ensureTokenInitialized(address token_) internal view {
333:        uint256 lastBlockAccrued = lastAccruedBlock[token_];  //@audit lastBlockAccrued variable unnecessary
334:
335:        if (lastBlockAccrued == 0) {
336:            revert TokenNotInitialized(token_);
337:        }
338:    }
```
In the `_ensureTokenInitialized()` function above a variable `lastBlockAccrued` was declared but was only used once in the function. We could use `lastAccruedBlock[token_]` directly in place of the `lastBlockAccrued` variable. The diff below shows how the code should be refactored:
```diff
diff --git a/contracts/Tokens/Prime/PrimeLiquidityProvider.sol b/contracts/Tokens/Prime/PrimeLiquidityProvider.sol
index 0cbb17a..f4a8bcf 100644
--- a/contracts/Tokens/Prime/PrimeLiquidityProvider.sol
+++ b/contracts/Tokens/Prime/PrimeLiquidityProvider.sol
@@ -330,9 +330,8 @@ contract PrimeLiquidityProvider is AccessControlledV8, PausableUpgradeable {
      * @param token_ Token Address to be verified for
      */
     function _ensureTokenInitialized(address token_) internal view {
-        uint256 lastBlockAccrued = lastAccruedBlock[token_];

-        if (lastBlockAccrued == 0) {
+        if (lastAccruedBlock[token_] == 0) {
             revert TokenNotInitialized(token_);
         }
     }
```
```
Estimated gas saved: 3 gas units.
```





## [G-08] Use += for mappings
Using += for mappings saves 40 gas due to not having to recalculate the mapping's value's hash.

### 3 Instances
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L588
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L633
- https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L800
 



## CONCLUSION
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.