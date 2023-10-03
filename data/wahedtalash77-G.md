|     |     |
| --- | --- |
| ID  | Description |
| \[G-01\] | Do not calculate constants |
| \[G‑02\] | Functions guaranteed to revert when called by normal users can be marked payable |
| \[G-03\] | use Mappings Instead of Arrays |
| \[G-04\] | Make fewer external calls. |
| \[G-05\] | Make 3 event parameters indexed when possible |
| \[G-06\] | Use assembly to validate `msg.sender` |
| \[G-07\] | Use hardcode address instead `address(this)` |
| \[G-08\] | Amounts should be checked for 0 before calling a transfer |
| \[G-09\] | IF’s/require() statements that check input arguments should be at the top of the function to save gas if the condition become `false/true` |
| \[G-10\] | Return values from external calls can be cached to avoid unnecessary call |
| \[G-11\] | When possible, use assembly instead of `unchecked{}` |
| \[G-12\] | For loops in public or external functions should be avoided due to high gas costs and possible DOS |
| \[G-13\] | Shorten the array rather than copying to a new one |
| \[G-14\] | Expensive operation inside a for loop |
| \[G‑15\] | Counting down in `for` statements is more gas efficient |
| **\[G-16\]** | Ternary operation is cheaper than if-else statement |
| \[G-17\] | The result of a function call should be cached rather than re-calling the function |
| \[G-18\] | `++i` costs less gas then `i++`, especially when it's used for-loop(`--i/i--`) |
| \[G-19\] | Use assembly for math (add, sub, mul, div) |
| \[  0 \] | CONCLUSION |

## \[G-01\] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```Solidity
uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;

    /// @notice maximum XVS taken in account when calculating user score
    uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

    /// @notice number of days user need to stake to claim prime token
    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L34C3-L40C64

## \[G‑02\] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

```Solidity
118: function initializeTokens(address[] calldata tokens_) external onlyOwner {
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118

```
177:    function setPrimeToken(address prime_) external onlyOwner {
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177

```
216: function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216

## \[G-03\]use Mappings Instead of Arrays

There are two data types to describe lists of data in Solidity, arrays and maps, and their syntax and structure are quite different, allowing each to serve a distinct purpose. While arrays are packable and iterable, mappings are less expensive.

```
// For example, creating an array of cars in Solidity might look like this:

// Let’s see how to create a mapping for cars:

mapping(uint => string) public cars

//When using the mapping keyword, you will specify the data type for the key (uint) and the value (string). Then you can add some data using the constructor function.

constructor() public { cars[101] = "Ford"; cars[102] = "Audi"; cars[103] = "Chevrolet"; } }
```

Except where iteration is required or data types can be packed, it is advised to use mappings to manage lists of data in order to conserve gas. This is beneficial for both memory and storage.

An integer index can be used as a key in a mapping to control an ordered list. Another advantage of mappings is that you can access any value without having to iterate through an array as would otherwise be necessary.

```
70:  address[] internal allMarkets;
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L70

```
124: uint256[25] private __gap;
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L124

## \[G-04\] Make fewer external calls.

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, It’s better to call one function and have it return all the data you need rather than calling a separate function for every piece of data. This might go against the best coding practices for other languages, but solidity is special.

## \[G-05\] Make 3 event parameters indexed when possible

events are used to emit information about state changes in a contract. When defining an event, it's important to consider the gas cost of emitting the event, as well as the efficiency of searching for events in the Ethereum blockchain.

Making event parameters indexed can help reduce the gas cost of emitting events and improve the efficiency of searching for events. When an event parameter is marked as indexed, its value is stored in a separate data structure called the event topic, which allows for more efficient searching of events.

```
event AlphaUpdated(
        uint128 indexed oldNumerator,
        uint128 indexed oldDenominator,
        uint128 indexed newNumerator,
        uint128 newDenominator
    );
    
    event MultiplierUpdated(
        address indexed market,
        uint256 indexed oldSupplyMultiplier,
        uint256 indexed oldBorrowMultiplier,
        uint256 newSupplyMultiplier,
        uint256 newBorrowMultiplier
    );
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L74C1-L88C7

```Solidity
/// @notice Emitted when prime token contract address is changed
    event PrimeTokenUpdated(address oldPrimeToken, address newPrimeToken);

    /// @notice Emitted when distribution state(Index and block) is updated
    event TokensAccrued(address indexed token, uint256 amount);

    /// @notice Emitted when token is transferred to the prime contract
    event TokenTransferredToPrime(address indexed token, uint256 amount);

    /// @notice Emitted on sweep token success
    event SweepToken(address indexed token, address indexed to, uint256 sweepAmount);

    /// @notice Emitted on updation of initial balance for token
    event TokenInitialBalanceUpdated(address indexed token, uint256 balance);
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L35C4-L48C78

## \[G-06\] Use assembly to validate `msg.sender`

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```
File : contracts/Tokens/Prime/Prime.sol

398:  if (stakedAt[msg.sender] == 0) revert IneligibleToClaim();

453: if (msg.sender != protocolShareReserve) revert InvalidCaller();
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

## \[G-07\] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

References: <ins>https://book.getfoundry.sh/reference/forge-std/compute-create-address</ins>

<ins>https://twitter.com/transmissions11/status/1518507047943245824</ins>

```Solidity
File : contracts/Tokens/Prime/Prime.sol

564: address(this),

682: if (amount > asset.balanceOf(address(this))) {

686 : if (amount > asset.balanceOf(address(this))) {

960:  address(this),
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

## \[G-08\] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas. While this is done at some places, it's not consistently done in the solution.

```Solidity
File : contracts/Tokens/Prime/Prime.sol

692: asset.safeTransfer(user, amount);
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L692

```
224: token_.safeTransfer(to_, amount_);
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L224

## \[G-09\] IF’s/require() statements that check input arguments should be at the top of the function to save gas if the condition become `false/true`

```Solidity
function _calculateUserAPR(
        address vToken,
        uint256 totalSupply,
        uint256 totalBorrow,
        uint256 totalCappedSupply,
        uint256 totalCappedBorrow,
        uint256 userScore,
        uint256 totalScore
    ) internal view returns (uint256 supplyAPR, uint256 borrowAPR) {
        if (totalScore == 0) return (0, 0);
 +	if (totalCappedValue == 0) return (0, 0);
        uint256 userYearlyIncome = (userScore * _incomeDistributionYearly(vToken)) / totalScore;
        uint256 totalCappedValue = totalCappedSupply + totalCappedBorrow;

 -       if (totalCappedValue == 0) return (0, 0);

        uint256 userSupplyIncomeYearly = (userYearlyIncome * totalCappedSupply) / totalCappedValue;
        uint256 userBorrowIncomeYearly = (userYearlyIncome * totalCappedBorrow) / totalCappedValue;

        supplyAPR = totalSupply == 0 ? 0 : ((userSupplyIncomeYearly * MAXIMUM_BPS) / totalSupply);
        borrowAPR = totalBorrow == 0 ? 0 : ((userBorrowIncomeYearly * MAXIMUM_BPS) / totalBorrow);
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L1002C8-L1013C99

## \[G-10\] Return values from external calls can be cached to avoid unnecessary call

External calls are expensive as they use the `STATICCALL`/`CALL` opcode (~100 gas). If you are calling the same external function more than once you should cache the return value to avoid an unnecessary `STATICCALL`/`CALL`.

## \[G-11\] When possible, use assembly instead of `unchecked{}`

You can also use `unchecked{}` for even more gas savings but this will not check to see if I overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

```Solidity
unchecked {
                ++i;
            }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L107C13-L109C14

```Solidity
unchecked {
                ++i;
            }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L122C12-L124C14

```Solidity
unchecked {
                ++i;
            }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L165

```Solidity
unchecked {
                i++;
            }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L249

```Solidity
unchecked {
                i++;
            }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L344

```Solidity
unchecked {
                i++;
            }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L354

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L613

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L635

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L739

## \[G-12\] For loops in public or external functions should be avoided due to high gas costs and possible DOS

```
function initialize(
        address accessControlManager_,
        address[] calldata tokens_,
        uint256[] calldata distributionSpeeds_
    ) external initializer {
        __AccessControlled_init(accessControlManager_);
        __Pausable_init();

        uint256 numTokens = tokens_.length;
        if (numTokens != distributionSpeeds_.length) {
            revert InvalidArguments();
        }

        for (uint256 i; i < numTokens; ) {
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L90C4-L105C76

```Solidity
function initializeTokens(address[] calldata tokens_) external onlyOwner {
        for (uint256 i; i < tokens_.length; ) {
            _initializeToken(tokens_[i]);

            unchecked {
                ++i;
            }
        }
    }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118C5-L126C6

```
function setTokensDistributionSpeed(address[] calldata tokens_, uint256[] calldata distributionSpeeds_) external {
        _checkAccessAllowed("setTokensDistributionSpeed(address[],uint256[])");
        uint256 numTokens = tokens_.length;

        if (numTokens != distributionSpeeds_.length) {
            revert InvalidArguments();
        }

        for (uint256 i; i < numTokens; ) {
            _ensureTokenInitialized(tokens_[i]);
            _setTokenDistributionSpeed(tokens_[i], distributionSpeeds_[i]);
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L153C4-L163C76

```Solidity
File : contracts/Tokens/Prime/Prime.sol

174:  function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {

200: function updateScores(address[] memory users) external {

237: function updateAlpha(uint128 _alphaNumerator, uint128 _alphaDenominator) external {

331: function issue(bool isIrrevocable, address[] calldata users) external {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

## \[G-13\] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don’t have to be copied to a new, shorter array.

```
176:  PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L176

## \[G-14\] Expensive operation inside a for loop

```
File : contracts/Tokens/Prime/Prime.sol

174:  function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
        address[] storage _allMarkets = allMarkets;
        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

        for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            uint256 interestAccrued = getInterestAccrued(market, user);
            uint256 accrued = interests[market][user].accrued;

            pendingInterests[i] = PendingInterest({
                market: IVToken(market).underlying(),
                amount: interestAccrued + accrued
            });

            unchecked {
                i++;
            }
        }
```

```Solidity
File : contracts/Tokens/Prime/Prime.sol

200: function updateScores(address[] memory users) external {

237: function updateAlpha(uint128 _alphaNumerator, uint128 _alphaDenominator) external {

331: function issue(bool isIrrevocable, address[] calldata users) external {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

## \[G‑15\] Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

> All Loops in Contests

## **\[G-16\]** Ternary operation is cheaper than if-else statement

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

```
if (paused()) {
            _unpause();
        } else {
            _pause();
        }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L421C8-L425C10

## \[G-17\] The result of a function call should be cached rather than re-calling the function

*External calls are expensive. External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following*

```Solidity
File:  contracts/Tokens/Prime/libs/FixedMath.sol

25:   return (n.toInt256() * FixedMath0x.FIXED_1) / int256(d.toInt256());

37:  return uint256((u.toInt256() * FixedMath0x.FIXED_1) / f);

49: return uint256((u.toInt256() * f) / FixedMath0x.FIXED_1);

54: return FixedMath0x.ln(x);

59:  return FixedMath0x.exp(x);
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol

## \[G-18\] `++i` costs less gas then `i++`, especially when it's used for-loop(`--i/i--`)

> all contests

## CONCLUSION

As you move forward with implementing the suggested optimizations, we urge you to exercise caution and conduct meticulous testing. It is essential to verify that these changes do not introduce any new vulnerabilities and effectively deliver the desired performance improvements. Carefully review the code modifications and perform rigorous testing to validate the security and effectiveness of the refactored code.