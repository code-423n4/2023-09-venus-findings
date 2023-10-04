## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Optimize Names to save Gas | 2 | - |
| [G-02] | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | 5 | - |
| [G-03] | Functions guaranteed to when called by normal users can be marked Payable | 3 | - |
| [G-04] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= ) | 1 | - |
| [G-05] | Can make the variable outside the loop to save gas | 6 | - |
| [G-06] | Using storage instead of memory for structs/arrays saves gas | 4 | - |
| [G-07] | Do not calculate  constant | 3 | - |
| [G-08] | Sort Solidity operations using short-circuit mode | 2 | - |
| [G-09] | Use hardcode address instead address(this) | 2 | - |
| [G-10] | array[index] += amount is cheaper than array[index] = array[index] + amount (or related variants) | 2 | - |
| [G-11] | Use assembly to validate msg.sender | 3 | - |
| [G-12] | Make 3 event parameters indexed when possible | 12 | - |
| [G-13] | Low level call can be optimized with assembly | 3 | - |

## Gas Optimizations  

## [G-01]  Optimize Names to save Gas

public/external function names and public member variable names can be optimized to save gas. See this link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted.

```solidity
file: /contracts/Tokens/Prime/Prime.sol

35    contract Prime is IIncomeDestination, AccessControlledV8, PausableUpgradeable, MaxLoopsLimitHelper, PrimeStorageV1 {

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L35


```solidity
file: /contracts/Tokens/Prime/PrimeLiquidityProvider.sol

8     contract PrimeLiquidityProvider is AccessControlledV8, PausableUpgradeable {

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L8

## [G-02] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas

```solidity
file: /contracts/Tokens/Prime/Prime.sol

710        if (isIrrevocable) {
            totalIrrevocable++;         //FOUND
         } else {
713            totalRevocable++;        //FOUND


766        totalIrrevocable++;         //FOUND

767        totalRevocable--;           //FOUND

819        nextScoreUpdateRoundId++;         //FOUND

828        if (totalScoreUpdatesRequired > 0) totalScoreUpdatesRequired--;         //FOUND

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L710-L713

### Recommended code

```soldity
       if (isIrrevocable) {
++    uncheked{
            totalIrrevocable++;
            }
         } else {
++            uncheked{
           totalRevocable++;
 }

```


## [G-03] Functions guaranteed to when called by normal users can be marked Payable

The onlyOwner modifier makes a function revert if not called by the address registered as the owner

```solidity
file: /contracts/Tokens/Prime/PrimeLiquidityProvider.sol

118    function initializeTokens(address[] calldata tokens_) external onlyOwner {

177    function setPrimeToken(address prime_) external onlyOwner {

216    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {        

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118


## [G-04] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= )

AVOID COMPOUND ASSIGNMENT OPERATOR IN STATE VARIABLES.
Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

```solidity
file: /contracts/Tokens/Prime/PrimeLiquidityProvider.sol

///@audit the ' tokenAmountAccrued ' is state var on line 27

266      tokenAmountAccrued[token_] += tokenAccrued;

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L266


## [G-05] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /contracts/Tokens/Prime/Prime.sol

181            address market = _allMarkets[i];

182            uint256 interestAccrued = getInterestAccrued(market, user);

183            uint256 accrued = interests[market][user].accrued;

210            address[] storage _allMarkets = allMarkets;

336            Token storage userToken = tokens[users[i]];

626            address market = _allMarkets[i];

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L179-L181


## [G-06] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
file: /contracts/Tokens/Prime/Prime.sol

///@audit ' PendingInterest[] memory pendingInterests ' instead of memory, should use storage

174    function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {

176        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

200    function updateScores(address[] memory users) external {

683            address[] memory assets = new address[](1);    

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174


## [G-07] Do not calculate  constant

When you define a constant in Solidity, the compiler can calculate its value at compile-time and replace all references to the constant with its computed value. This can be helpful for readability and reducing the size of the compiled code, but it can also increase gas usage at runtime.

```solidity
file: /contracts/Tokens/Prime/PrimeStorage.sol

34    uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;

37    uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

40    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L34-L40


## [G-08] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows
 f(x) || g(y) 
 f(x) && g(y)
```

```solidity
file: /contracts/Tokens/Prime/Prime.sol

369        if (tokens[user].exists && !isAccountEligible) {

379        } else if (tokens[user].exists && isAccountEligible) {    

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L369


## [G-09] Use hardcode address instead address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

an example :
```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```

```solidity
file: /contracts/Tokens/Prime/Prime.sol

682        if (amount > asset.balanceOf(address(this))) {

686            if (amount > asset.balanceOf(address(this))) {    

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L682


## [G-10]  array[index] += amount is cheaper than array[index] = array[index] + amount (or related variants)

When updating a value in an array with arithmetic, using array[index] += amount is cheaper than array[index] = array[index] + amount. This is because you avoid an additonal mload when the array is stored in memory, and an sload when the array is stored in storage. This can be applied for any arithmetic operation including +=, -=,/=,*=,^=,&=, %=, <<=,>>=, and >>>=. This optimization can be particularly significant if the pattern occurs during a loop.

```solidity
file: /contracts/Tokens/Prime/Prime.sol

633            markets[market].sumOfMembersScore = markets[market].sumOfMembersScore + score;

800        markets[market].sumOfMembersScore = markets[market].sumOfMembersScore - interests[market][user].score + score;

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L633


### Recommended code

```solidity

++           markets[market].sumOfMembersScore += score;

++      markets[market].sumOfMembersScore -= interests[market][user].score + score;


```

## [G-11] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file: /contracts/Tokens/Prime/Prime.sol

398        if (stakedAt[msg.sender] == 0) revert IneligibleToClaim();

399        if (block.timestamp - stakedAt[msg.sender] < STAKING_PERIOD) revert WaitMoreTime();

434        return _claimInterest(vToken, msg.sender);

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L398


## [G-12] Make 3 event parameters indexed when possible

It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file: /contracts/Tokens/Prime/Prime.sol

57    event UpdatedAssetsState(address indexed comptroller, address indexed asset);

91    event InterestClaimed(address indexed user, address indexed market, uint256 amount);

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L57


```solidity 
file: /contracts/Tokens/Prime/PrimeLiquidityProvider.sol
  
45    event SweepToken(address indexed token, address indexed to, uint256 sweepAmount);

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L45


## [G-13] Low level call can be optimized with assembly     

When using low-level calls, the returnData is copied to memory even if the variable is not utilized. The proper way to handle this is through a low level assembly call. For example:

```solidity
(bool success,) = payable(receiver).call{gas: gas, value: value}("");

can be optimized to:
bool success;
assembly {
    success := call(gas, receiver, value, 0, 0, 0, 0)
}

```

```solidity
file: /contracts/Tokens/Prime/Prime.sol

292        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);

561        uint256 totalIncomeUnreleased = IProtocolShareReserve(protocolShareReserve).getUnreleasedFunds(

571        uint256 totalAccruedInPLP = _primeLiquidityProvider.tokenAmountAccrued(underlying);    

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L292