     # gas optimization

# summary
|      |  issue  |  insteance  |
|------|---------|-------------|
|[G-01]|Avoid emitting storage values|5|
|[G-02]|Use Modifiers Instead of Functions To Save Gas|1|
|[G-03]|Cache external calls outside of loop to avoid re-calling function on each iteration|1|
|[G-04]|Do not calculate constants|3|
|[G-05]| Use hardcode address instead address(this)|7|
|[G-06]| Duplicated require()/if() checks should be refactored to a modifier or function|4|
|[G-07]|Stack variable used as a cheaper cache for a state variable is only used once|5|
|[G-08]|++i Costs Less Gas Than I++, Especially When It’s Used In For-loops (--i/i-- Too) This Will Catch My C4|19|
|[G-09]|Can Make The Variable Outside The Loop To Save Gas|4|
|[G-10]|Amounts should be checked for 0 before calling a transfer|3|
|[G-11]|Refactor structs to avoid using unnecessary storage slot|3|
|[G-12]|use Mappings Instead of Arrays|1|
|[G-13]|Replace state variable reads and writes within loops with local variable reads and writes.|12|
|[G-14]| Functions guaranteed to revert when called by normal users can be marked payable|3|
|[G-15]|State variables can be packed to use fewer storage slots|1|



## [G-01] Avoid emitting storage values

Emitting storage values refers to updating or modifying the state variables stored in the blockchain's storage during a transaction. While it is sometimes necessary to update storage values, emitting unnecessary storage updates can lead to higher gas costs in Ethereum and similar blockchain networks.

There is 5 instance of this issue:

```solidity
File:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol
// @audit state varible is prime
180   emit PrimeTokenUpdated(prime, prime_);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L180

```solidity
File:  contracts/Tokens/Prime/Prime.sol
// @audit state varible is alphaNumerator,alphaDenominator
241    emit AlphaUpdated(alphaNumerator, alphaDenominator, _alphaNumerator, _alphaDenominator);

// @audit state varible is markets[market]
269     emit MultiplierUpdated(
           market,
           markets[market].supplyMultiplier,
           markets[market].borrowMultiplier,
           supplyMultiplier,
           borrowMultiplier
       );


// @audit state varible is irrevocableLimit,revocableLimit
320   emit MintLimitsUpdated(irrevocableLimit, revocableLimit, _irrevocableLimit, _revocableLimit);   

// @audit state varible is comptroller
462   emit UpdatedAssetsState(comptroller, asset);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L241
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L269
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L320
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L462



## [G-02] Use Modifiers Instead of Functions To Save Gas


There is 1 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
904   function isEligible(uint256 amount) internal view returns (bool) {
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L904




## [G-03] Cache external calls outside of loop to avoid re-calling function on each iteration


Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

There is 1 instance of this issue:
```solidity
184   market: IVToken(market).underlying(),
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L184

## [G-04] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

There is 3 instance of this issue:
```solidity
   uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;

   /// @notice maximum XVS taken in account when calculating user score
   uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

   /// @notice number of days user need to stake to claim prime token
   uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L39-L40

## [G-05] Use hardcode address instead address(this)

it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.



There is 7 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
564   address(this),

682   if (amount > asset.balanceOf(address(this))) {

686   if (amount > asset.balanceOf(address(this))) {

960   address(this),       
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L564
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L682
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L686
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L960



```solidity
File:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol
217  uint256 balance = token_.balanceOf(address(this));

234  uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

259  uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L217
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L234
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L259




## [G-06] Duplicated require()/if() checks should be refactored to a modifier or function


Duplicating require() or if() checks throughout a smart contract can lead to increased gas costs. Refactoring these duplicated checks into modifiers or functions can help save gas.


```solidity
// @audit this if is also in line:726
207   if (!tokens[user].exists) revert UserHasNoPrimeToken();


// @audit this if is also in line:795
780   if (!markets[vToken].exists || !tokens[user].exists) {
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L207
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L780



```solidity
File:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol
// @audit this if is also in line:157
99   if (numTokens != distributionSpeeds_.length) {
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L99


```solidity
File:  contracts/Tokens/Prime/libs/FixedMath.sol
// @audit this if is also in line:47
35  if (f < 0) revert InvalidFixedPoint();
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L35

## [G‑07] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it's cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend

```solidity
// @audit distributionSpeed vriable is only access in line:238
233  uint256 distributionSpeed = tokenDistributionSpeeds[token_];

// @audit accrued vriable is only access in line:237
235   uint256 accrued = tokenAmountAccrued[token_];

// @audit accrued vriable is only access in line:335
333   uint256 lastBlockAccrued = lastAccruedBlock[token_];
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L233
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L235
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L333


```solidity
// @audit userScore and vriables only use once in line:514
503   uint256 userScore = interests[market][user].score;
504   uint256 totalScore = markets[market].sumOfMembersScore;

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L503-L503

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L503-L504




## [G‑08] ++i Costs Less Gas Than I++, Especially When It’s Used In For-loops (--i/i-- Too) This Will Catch My C4

```solidity
221    pendingScoreUpdates--;

711    totalIrrevocable++;

713    totalRevocable++;

745    totalIrrevocable--;

747    totalRevocable--;

766    totalIrrevocable++;
767    totalRevocable--;

819    nextScoreUpdateRoundId++;

828    totalScoreUpdatesRequired--;

831    pendingScoreUpdates--;

// @audit in these line of code also 189,217,225,250,345,355,614,636,740
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L221





## [G-09] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas 
```solidity
File:  contracts/Tokens/Prime/Prime.sol
179         address market = _allMarkets[i];
           uint256 interestAccrued = getInterestAccrued(market, user);
           uint256 accrued = interests[market][user].accrued;

205         address user = users[i];

212         address market = _allMarkets[j];

626         address market = _allMarkets[i];
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L179-L182
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L205
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L212
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L626


## [G-10] Amounts should be checked for 0 before calling a transfer

It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.



There is 3 instance of this issue:
```solidity
692   asset.safeTransfer(user, amount);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L692


```solidity
204   IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);

224   token_.safeTransfer(to_, amount_);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L204
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L224




## [G-11] Refactor structs to avoid using unnecessary storage slot

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

```solidity
   struct Market {
       uint256 supplyMultiplier;
       uint256 borrowMultiplier;
       uint256 rewardIndex;
       uint256 sumOfMembersScore;
       bool exists;
   }

   struct Interest {
       uint256 accrued;
       uint256 score;
       uint256 rewardIndex;
   }

   struct PendingInterest {
       address market;
       uint256 amount;
   }
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L12-L29

the above spend 10 storage slots but this fix code spend 6 storage slots

`Fix code`
```solidity
struct market {
   bool exists;
   uint128 supplymultiplier;
   uint128 borrowmultiplier;
   uint128 rewardindex;
   uint128 sumofmembersscore;
}

struct interest {
   uint128 rewardindex;
   uint128 accrued;
   uint128 score;
}

struct pendinginterest {
   address market;
   uint128 amount;
}

```

## [G-12] use Mappings Instead of Arrays

Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.




```solidity
File:  contracts/Tokens/Prime/PrimeStorage.sol
70    address[] internal allMarkets;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L70




## [G-13] Replace state variable reads and writes within loops with local variable reads and writes.

Accessing state variables in Ethereum incurs a gas cost. When state variable reads or writes are performed within loops, the gas cost accumulates with each iteration. By replacing these state variable accesses with local variable accesses, you eliminate the need for costly state variable operations within the loop, resulting in gas savings.

```solidity
function badCode() external {               
 for(uint256 i; i < myArray.length; i++) { // state reads
   myCounter++; // state reads and writes
 }       
}
```

```solidity
function goodCode() external {
   uint256 length = myArray.length; // one state read
   uint256 local_mycounter = myCounter; // one state read
   for(uint256 i; i < length; i++) { // local reads
       local_mycounter++; // local reads and writes 
   }
   myCounter = local_mycounter; // one state write
}

```
There is 12 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
// @audit _allMarkets is state reads
178  for (uint256 i = 0; i < _allMarkets.length; ) {

// @audit _allMarkets is state reads
211  for (uint256 j = 0; j < _allMarkets.length; ) {

// @audit allMarkets is state reads
246  for (uint256 i = 0; i < allMarkets.length; ) {

// @audit allMarkets is state reads
247  accrueInterest(allMarkets[i]);

// @audit stakedAt is state reads
352  delete stakedAt[users[i]];

// @audit _allMarkets is state reads
609  for (uint256 i = 0; i < _allMarkets.length; ) {
     _executeBoost(user, _allMarkets[i]);
     _updateScore(user, _allMarkets[i]);

// @audit _allMarkets is state reads
625  for (uint256 i = 0; i < _allMarkets.length; ) {
     address market = _allMarkets[i]; 

// @audit _allMarkets is state reads
730  for (uint256 i = 0; i < _allMarkets.length; ) {
      _executeBoost(user, _allMarkets[i]);         
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L178
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L211

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L247

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L352

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L609

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L625





## [G-14] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking thefunction as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a paymentwas provided.

```solidity
118    function initializeTokens(address[] calldata tokens_) external onlyOwner {

177    function setPrimeToken(address prime_) external onlyOwner {

216    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {       
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216


## [G-15] State variables can be packed to use fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).


```solidity
   uint256 public constant MAX_DISTRIBUTION_SPEED = 1e18;

   /// @notice exp scale
   uint256 internal constant EXP_SCALE = 1e18;

   /// @notice Address of the Prime contract
   address public prime;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L12-L18

above code spend 3 slots in EVM because each 32 bytes or 256 bites spend one slots in EVM
but you can reduce bites of uint256 to uint128 when you reduce to 128 now EVM spend 2 slots
because
```
uint256 public constant MAX_DISTRIBUTION_SPEED = 1e18;
```
spend one slot and also
``` 
  /// @notice exp scale
   uint128 internal constant EXP_SCALE = 1e18;

   /// @notice Address of the Prime contract
   address public prime;

```
this two also one slot

`Fix code`
```solidity
   uint256 public constant MAX_DISTRIBUTION_SPEED = 1e18;

   /// @notice exp scale
   uint128 internal constant EXP_SCALE = 1e18;

   /// @notice Address of the Prime contract
   address public prime;
```
