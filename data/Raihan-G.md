# gas optimization

# summary
|      |  issue  |  insteance  |
|------|---------|-------------|
|[G-01]|Can Make The Variable Outside The Loop To Save Gas|6|
|[G-02]|Amounts should be checked for 0 before calling a transfer|3|
|[G-03]|Use assembly to perform efficient back-to-back calls|5|
|[G-04]|Functions guaranteed to revert when called by normal users can be marked payable|3|
|[G-05]|State variables can be packed to use fewer storage slots|1|
|[G-06]|Refactor structs to avoid using unnecessary storage slot|3|
|[G-07]|use Mappings Instead of Arrays|1|
|[G-08]|Duplicated require()/if() checks should be refactored to a modifier or function|4|
|[G-09]|Stack variable used as a cheaper cache for a state variable is only used once|5|
|[G-10]|++i Costs Less Gas Than I++, Especially When It’s Used In For-loops (--i/i-- Too) This Will Catch My C4|19|
|[G-11]|Avoid emitting storage values|5|
|[G-12]|Use Modifiers Instead of Functions To Save Gas|1|
|[G-13]|Replace state variable reads and writes within loops with local variable reads and writes.|12|
|[G-14]|Cache external calls outside of loop to avoid re-calling function on each iteration|1|
|[G-15]|Do not calculate constants|3|
|[G-16]|Use hardcode address instead address(this)|7|
|[G-17]|Use assembly for loops|2|
|[G-18]|The creation of an intermediary array can be avoided|3|
|[G-19]|Using XOR (^) and AND (&) bitwise equivalents|3|
|[G-20]|Use assembly for math (add, sub, mul, div)|2|
|[G-21]|When possible, use assembly instead of unchecked{++i}|11|

# Details

## [G-01] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

There is 6 instance of this issue:
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


## [G-02] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.


There is 3 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
692   asset.safeTransfer(user, amount);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L692


```solidity
File:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol
204   IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);

224   token_.safeTransfer(to_, amount_);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L204


## [G-03] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)

There is 5 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
498     uint256 borrow = vToken.borrowBalanceStored(user);
        uint256 exchangeRate = vToken.exchangeRateStored();
        uint256 balanceOfAccount = vToken.balanceOf(user);

570     _primeLiquidityProvider.accrueTokens(underlying);
571     uint256 totalAccruedInPLP = _primeLiquidityProvider.tokenAmountAccrued(underlying);

651     uint256 borrow = vToken.borrowBalanceStored(user);
        uint256 exchangeRate = vToken.exchangeRateStored();
        uint256 balanceOfAccount = vToken.balanceOf(user);

657     oracle.updateAssetPrice(xvsToken);
658     oracle.updatePrice(market);     

682     if (amount > asset.balanceOf(address(this))) {
            address[] memory assets = new address[](1);
            assets[0] = address(asset);
            IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
            if (amount > asset.balanceOf(address(this))) {
                IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
                unreleasedPLPIncome[underlying] = 0;
            }
        }

692     asset.safeTransfer(user, amount);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L498-L500



## [G-04] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking thefunction as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a paymentwas provided.

There is 3 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol
118    function initializeTokens(address[] calldata tokens_) external onlyOwner {

177    function setPrimeToken(address prime_) external onlyOwner {

216    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {        
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118

## [G-05] State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

There is 1 instance of this issue:

```solidity
File:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol
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

## [G-06] Refactor structs to avoid using unnecessary storage slot
Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There is 3 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/PrimeStorage.sol
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

## [G-07] use Mappings Instead of Arrays
Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

`Note`: if it's posilbe.

There is 1 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/PrimeStorage.sol
70    address[] internal allMarkets;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L70


## [G-08] Duplicated require()/if() checks should be refactored to a modifier or function
sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.

Recommendation
You can consider adding a modifier like below
```
 modifer check (address checkToAddress) {    
     require(checkToAddress != address(0) && checkToAddress != SENTINEL_MODULES, "BSA101");  
      _; 
 }
```
There is 4 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
// @audit this if is also in line:726 
207   if (!tokens[user].exists) revert UserHasNoPrimeToken();


// @audit this if is also in line:795
780   if (!markets[vToken].exists || !tokens[user].exists) {
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L207


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

## [G‑09] Stack variable used as a cheaper cache for a state variable is only used once
If the variable is only accessed once, it's cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend

There is 5 instance of this issue:
```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol
// @audit distributionSpeed vriable is only access in line:238
233  uint256 distributionSpeed = tokenDistributionSpeeds[token_];

// @audit accrued vriable is only access in line:237
235   uint256 accrued = tokenAmountAccrued[token_];

// @audit accrued vriable is only access in line:335
333   uint256 lastBlockAccrued = lastAccruedBlock[token_];
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L233

```solidity
File:  contracts/Tokens/Prime/Prime.sol
// @audit userScore and vriables only use once in line:514
503   uint256 userScore = interests[market][user].score;
504   uint256 totalScore = markets[market].sumOfMembersScore;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L503-L504



## [G‑10] ++i Costs Less Gas Than I++, Especially When It’s Used In For-loops (--i/i-- Too) This Will Catch My C4

There is 19 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
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

## [G-11] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

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


## [G-12] Use Modifiers Instead of Functions To Save Gas
Example of two contracts with modifiers and internal view function:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}
```
Differences:
```
Deploy Modifier.sol
108727
Deploy Inlined.sol
110473
Modifier.foo
21532
Inlined.foo
21556
```
This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.

There is 1 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
904   function isEligible(uint256 amount) internal view returns (bool) {
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L904

## [G-13] Replace state variable reads and writes within loops with local variable reads and writes.
Reading and writing local variables is cheap, whereas reading and writing state variables that are stored in contract storage is expensive.

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

## [G-14] Cache external calls outside of loop to avoid re-calling function on each iteration
Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

There is 1 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
184   market: IVToken(market).underlying(),
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L184

## [G-15] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas
[Reffrence](https://code4rena.com/reports/2022-07-golom#g-24-do-not-calculate-constants)

There is 3 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/PrimeStorage.sol
    uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;

    /// @notice maximum XVS taken in account when calculating user score
    uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

    /// @notice number of days user need to stake to claim prime token
    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L39-L40

## [G-16] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    #L
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

There is 7 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
564   address(this),

682   if (amount > asset.balanceOf(address(this))) {

686   if (amount > asset.balanceOf(address(this))) {

960   address(this),        
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L564


```solidity
File:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol
217  uint256 balance = token_.balanceOf(address(this));

234  uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

259  uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L217

## [G-17] Use assembly for loops
In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

Note that in order to do this optimization safely we will need to cache and restore the free memory pointer after the loop. We will also set the zero slot (0x60) back to 0.

There is 2 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
246  for (uint256 i = 0; i < allMarkets.length; ) {
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246


```solidity
File:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol
119  for (uint256 i; i < tokens_.length; ) {
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L119


## [G-18] The creation of an intermediary array can be avoided
Sometimes, it’s not necessary to create an intermediate array to store values.
In this case, an array of maximum size is created because we don’t yet know what size the final array will be. This is not useful, as it’s more efficient to keep this maximum-size array, fill it and then reduce its size using assembly.

[Reffrence](https://code4rena.com/reports/2023-08-arbitrum#g-02-the-creation-of-an-intermediary-array-can-be-avoided)

There is 3 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
200  function updateScores(address[] memory users) external {
        if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
        if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

        for (uint256 i = 0; i < users.length; ) {

331    function issue(bool isIrrevocable, address[] calldata users) external {
        _checkAccessAllowed("issue(bool,address[])");

        if (isIrrevocable) {
            for (uint256 i = 0; i < users.length; ) {            
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L204


```solidity
File:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol
118   function initializeTokens(address[] calldata tokens_) external onlyOwner {
        for (uint256 i; i < tokens_.length; ) {
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118-L119

## [G‑19] Using XOR (^) and AND (&) bitwise equivalents
Given 4 variables a, b, c and d represented as such:
```
0 0 0 0 0 1 1 0 <- a
0 1 1 0 0 1 1 0 <- b
0 0 0 0 0 0 0 0 <- c
1 1 1 1 1 1 1 1 <- d
```

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities. Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

There is 3 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
1002  if (totalScore == 0) return (0, 0);

1007  if (totalCappedValue == 0) return (0, 0);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L1002


```solidity
File:  contracts/Tokens/Prime/libs/FixedMath0x.sol
145   if (x == 0) {
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L145

## [G-20] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

Addition:
```
//addition in SolidityCache multiple accesses of a mapping/array
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```
Gas: 303
```
//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263


Subtraction:
```
//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}

```
Gas: 300
```
//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263

Multiplication:
```
//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325
```
//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265

Division
```
//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325

```
//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265

There is 2 instance of this issue:
```solidity
File:  contracts/Tokens/Prime/Prime.sol
483  return STAKING_PERIOD - totalTimeStaked;

846  return (xvs - pendingWithdrawals);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L483

## [G-21] When possible, use assembly instead of unchecked{++i}
You can also use unchecked{++i;} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

There is 11 instance of this issue:
```
//loop with unchecked{++i}
function uncheckedPlusPlusI() public pure {
    uint256 j = 0;
    for (uint256 i; i < 10; ) {
        j++;
        unchecked {
            ++i;
        }
    }
}
```
Gas: 1329
```
//loop with inline assembly
function inlineAssemblyLoop() public pure {
    assembly {
        let j := 0
        for {
            let i := 0
        } lt(i, 10) {
            i := add(i, 0x01)
        } {
            j := add(j, 0x01)
        }
    }
}
```
Gas: 709

```solidity
File:  contracts/Tokens/Prime/Prime.sol
188         unchecked {
                i++;
            }
// @audit in these line of code also 224,249,344,354,613,635,739
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L188

```solidity
File:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol
107         unchecked {
                ++i;
            }

122         unchecked {
                ++i;
            }        

165         unchecked {
                ++i;
            }                
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L107
