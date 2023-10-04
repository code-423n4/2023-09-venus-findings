## [G-01] The result of a function call should be cached rather than re-calling the function

*External calls are expensive. External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following*

```
File:  contracts/Tokens/Prime/libs/FixedMath.sol

25:   return (n.toInt256() * FixedMath0x.FIXED_1) / int256(d.toInt256());

37:  return uint256((u.toInt256() * FixedMath0x.FIXED_1) / f);

49: return uint256((u.toInt256() * f) / FixedMath0x.FIXED_1);

54: return FixedMath0x.ln(x);

59:  return FixedMath0x.exp(x);
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol

## [G-02] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```Solidity
uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;

    /// @notice maximum XVS taken in account when calculating user score
    uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

    /// @notice number of days user need to stake to claim prime token
    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L34C3-L40C64

## [G‑03] Functions guaranteed to revert when called by normal users can be marked payable

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

## [G-04] Make 3 event parameters indexed when possible

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

## [G-05] When possible, use assembly instead of `unchecked{}`

You can also use `unchecked{}` for even more gas savings but this will not check to see if I overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

```
unchecked {
                i++;
            }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L107C13-L109C14

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L122C12-L124C14

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L165

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L249

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L344

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L354

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L613

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L635

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L739

## [G‑06] Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

## **[G-07]** Ternary operation is cheaper than if-else statement

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

```
if (paused()) {
            _unpause();
        } else {
            _pause();
        }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L421C8-L425C10

## [G-08] `++i` costs less gas then `i++`, especially when it's used for-loop(`--i/i--`)

## [G-09]use Mappings Instead of Arrays

```
70:  address[] internal allMarkets;
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L70

```
124: uint256[25] private __gap;
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L124