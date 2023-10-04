
### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
|[G-1] Avoid unnecessary storage updates | [Avoid unnecessary storage updates](#avoid-unnecessary-storage-updates) | 3 | 2400 |
|[G-2] Multiplication and Division by 2 Should use in Bit Shifting | [Multiplication and Division by 2 Should use in Bit Shifting](#multiplication-and-division-by-2-should-use-in-bit-shifting) | 5 | 100 |
|[G-3] Modulus operations that could be unchecked | [Modulus operations that could be unchecked](#modulus-operations-that-could-be-unchecked) | 1 | 85 |
|[G-4] Inefficient Parameter Storage | [Inefficient Parameter Storage](#inefficient-parameter-storage) | 1 | 50 |
|[G-5] Short-circuit rules can be used to optimize some gas usage | [Short-circuit rules can be used to optimize some gas usage](#short-circuit-rules-can-be-used-to-optimize-some-gas-usage) | 3 | 6300 |
|[G-6] Unnecessary Casting of Variables | [Unnecessary Casting of Variables](#unnecessary-casting-of-variables) | 1 | - |
|[G-7] Unused Named Return Variables | [Unused Named Return Variables](#unused-named-return-variables) | 5 | - |

Total:  7 issues

#

## Avoid unnecessary storage updates
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 2400


### Note 
I reported only on three issues that were missing in the wining bot.

### Description
Avoid updating storage when the value hasn't changed. If the old value is equal to the new value, not re-storing the value will avoid a `SSTORE` operation (costing 2900 gas), potentially at the expense of a `SLOAD` operation (2100 gas) or a `WARMACCESS` operation (100 gas).

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#
The function `updateAlpha()` changes the state variable without first verifying if the values are different.


```
File: contracts/Tokens/Prime/Prime.sol

243    alphaNumerator = _alphaNumerator
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L243](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L243)


#
The function `updateAlpha()` changes the state variable without first verifying if the values are different.


```
File: contracts/Tokens/Prime/Prime.sol

244    alphaDenominator = _alphaDenominator
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L244](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L244)


#
The function `updateAssetsState()` changes the state variable without first verifying if the values are different.


```
File: contracts/Tokens/Prime/Prime.sol

460    unreleasedPSRIncome[_getUnderlying(address(market))] = 0
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L460](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L460)


</details>

# 



## Multiplication and Division by 2 Should use in Bit Shifting
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 100

### Description
The expressions 'x * 2' and 'x / 2' can be optimized for gas efficiency by utilizing bitwise operations. In Solidity, you can achieve the same results by using bitwise left shift (x << 1) for multiplication and bitwise right shift (x >> 1) for division.

Using bitwise shift operations (SHL and SHR) instead of multiplication (MUL) and division (DIV) opcodes can lead to significant gas savings. The MUL and DIV opcodes cost 5 gas, while the SHL and SHR opcodes incur a lower cost of only 3 gas.

By leveraging these more efficient bitwise operations, you can reduce the gas consumption of your smart contracts and enhance their overall performance.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

122    r += (z * (0x100000000000000000000000000000000 - y)) / 0x100000000000000000000000000000000
```
 instead `340282366920938463463374607431768211456` use bit shifting `128` 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L122](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L122)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

124    r += (z * (0x0aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa - y)) / 0x200000000000000000000000000000000
```
 instead `680564733841876926926749214863536422912` use bit shifting `129` 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L124](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L124)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

128    r += (z * (0x092492492492492492492492492492492 - y)) / 0x400000000000000000000000000000000
```
 instead `1361129467683753853853498429727072845824` use bit shifting `130` 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L128](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L128)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

136    r += (z * (0x088888888888888888888888888888888 - y)) / 0x800000000000000000000000000000000
```
 instead `2722258935367507707706996859454145691648` use bit shifting `131` 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L136](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L136)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

200    r += z * 0x0000000000000001
```
 instead `1` use bit shifting `0` 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L200](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L200)


</details>

# 


## Modulus operations that could be unchecked
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 85

### Description
Modulus operations should be unchecked to save gas since they cannot overflow or underflow. Execution of modulus operations outside `unchecked` blocks adds nothing but overhead. Saves about 30 gas.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

162    x % 0x0000000000000000000000000000000010000000000000000000000000000000
```
 should be unchecked

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L162](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L162)


</details>

# 


## Short-circuit rules can be used to optimize some gas usage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 6300

### Description
Some conditions may be reordered to save an SLOAD (2100 gas), as we avoid reading state variables when the first part of the condition fails (with &&), or succeeds (with ||). For instance, consider a scenario where you have a `stateVariable` (a variable stored in contract storage) and a `localVariable` (a variable in memory). 

If you have a condition like `stateVariable > 0 && localVariable > 0`, if `localVariable > 0` is false, the Solidity runtime will still execute `stateVariable > 0`, which costs an SLOAD operation (2100 gas). However, if you reorder the condition to `localVariable > 0 && stateVariable > 0`, the `stateVariable > 0` check won't happen if `localVariable > 0` is false, saving you the SLOAD gas cost.

Similarly, for the `||` operator, if you have a condition like `stateVariable > 0 || localVariable > 0`, and `stateVariable > 0` is true, the Solidity runtime will still execute `localVariable > 0`. But if you reorder the condition to `localVariable > 0 || stateVariable > 0`, and `localVariable > 0` is true, the `stateVariable > 0` check won't happen, again saving you the SLOAD gas cost.

This detector checks for such conditions in the contract and reports if any condition could be optimized by taking advantage of the short-circuiting behavior of `&&` and `||`.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

377    stakedAt[user] == 0 && isAccountEligible && !tokens[user].exists
```


```
 // @audit: Switch isAccountEligible && stakedAt[user] == 0 
```
[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L377](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L377)


#

```
File: contracts/Tokens/Prime/Prime.sol

379    tokens[user].exists && isAccountEligible
```


```
 // @audit: Switch isAccountEligible && tokens[user].exists 
```
[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L379](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L379)


#

```
File: contracts/Tokens/Prime/Prime.sol

369    tokens[user].exists && !isAccountEligible
```


```
 // @audit: Switch ! isAccountEligible && tokens[user].exists 
```
[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L369](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L369)


</details>

# 


## Unnecessary Casting of Variables
- Severity: Gas Optimization
- Confidence: High

### Description
This detector scans for instances where a variable is casted to its own type. This is unnecessary and can be safely removed to improve code readability.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/libs/FixedMath.sol

25    return (n.toInt256() * FixedMath0x.FIXED_1) / int256(d.toInt256())
```
Unnecessary cast: `int256(d.toInt256())` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L25](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L25)


</details>

# 


## Unused Named Return Variables
- Severity: Gas Optimization
- Confidence: High

### Description
Named return variables allow for clear and explicit naming of values to be returned from a function. However, when these variables are unused, it can lead to confusion and make the code less maintainable.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

174    function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) 
```
there is not use of this variables:
@ **pendingInterests**

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L194](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L194)


#

```
File: contracts/Tokens/Prime/Prime.sol

496    function calculateAPR(address market, address user) external view returns (uint256 supplyAPR, uint256 borrowAPR) 
```
there is not use of this variables:
@ **supplyAPR**
@ **borrowAPR**

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496-L515](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496-L515)


#

```
File: contracts/Tokens/Prime/Prime.sol

527    function estimateAPR(
528            address market,
529            address user,
530            uint256 borrow,
531            uint256 supply,
532            uint256 xvsStaked
533        ) external view returns (uint256 supplyAPR, uint256 borrowAPR) 
```
there is not use of this variables:
@ **supplyAPR**
@ **borrowAPR**

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548)


#

```
File: contracts/Tokens/Prime/libs/FixedMath.sol

53    function ln(int256 x) internal pure returns (int256 r) 
```
there is not use of this variables:
@ **r**

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L53-L55](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L53-L55)


#

```
File: contracts/Tokens/Prime/libs/FixedMath.sol

58    function exp(int256 x) internal pure returns (int256 r) 
```
there is not use of this variables:
@ **r**

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L58-L60](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L58-L60)


</details>

# 



## Inefficient Parameter Storage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 50

### Description
When passing function parameters, using the `calldata` area instead of `memory` can improve gas efficiency. Calldata is a read-only area where function arguments and external function calls' parameters are stored.

By using `calldata` for function parameters, you avoid unnecessary gas costs associated with copying data from `calldata` to memory. This is particularly beneficial when the parameter is read-only and doesn't require modification within the contract.

Using `calldata` for function parameters can help optimize gas usage, especially when making external function calls or when the parameter values are provided externally and don't need to be stored persistently within the contract.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

200    address[] memory users
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200)


</details>

# 