| | issue |
| ----------- | ----------- |
| 1 | [state variables should be cached in stack variables rather than re-reading them from storage](#1-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage-ones-not-mentioned-in-bot-report) |
| 2 | [Stack variable used as a cheaper cache for a state variable is only used once](#2-state-variable-is-already-defaulted-to-zero-when-defined-there-is-no-need-to-assign-it-to-zero-again) |
| 3 | [Avoid updating storage when the value hasn't changed (ones not mentioned in bot report)](#3-avoid-updating-storage-when-the-value-hasnt-changed-ones-not-mentioned-in-bot-report) |
| 4 | [`++i` costs less gas than `i++`, especially when it’s used in for-loops (--i/i-- too)](#4-i-costs-less-gas-than-i-especially-when-its-used-in-for-loops---ii---too) |
| 5 | [it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied](#5-it-costs-more-gas-to-initialize-non-constantnon-immutable-variables-to-zero-than-to-let-the-default-of-zero-be-applied-not-in-the-bot-report) |
| 6 | [ Ternary over if ... else](#6-ternary-over-if--else) |
| 7 | [before some functions we should check some variables for possible gas save](#7-before-some-functions-we-should-check-some-variables-for-possible-gas-save) |
| 8 | [array[index] += amount is cheaper than array[index] = array[index] + amount (or related variants)](#8-arrayindex--amount-is-cheaper-than-arrayindex--arrayindex--amount-or-related-variants) |
| 9 | [Optimize names to save gas](#9-optimize-names-to-save-gas) |
| 10 | [Use hardcoded address instead address(this)](#10-use-hardcoded-address-instead-addressthis) |
| 11 | [Initializers can be marked as payable to save deployment gas](#11-initializers-can-be-marked-as-payable-to-save-deployment-gas) |

## 1. state variables should be cached in stack variables rather than re-reading them from storage (ones not mentioned in bot report)

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

`nextScoreUpdateRoundId` value never changes in `updateScores` function and its being read once before the loop and twice inside the loop, if we cache this state variable before #L202 and use this cached version in all the places currently used in the function this will result in 200*iterations gas save.()

- [Prime.sol#L208](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L208)

`allMarkets` is gonna be read lots of times (1 + loop iterations time) which is very expensive, we should cache the values of the list or make a pointer to it as it has been done in other functions beforehand to save lots of gas. (a sample of the better way in the code is in #L210-L212)

- [Prime.sol#L246](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246)

instead of putting `totalIrrevocable + totalRevocable` result inside the `totalScoreUpdatesRequired` state variable in one go, first put it in a stack variable and then assign that stack var to the `totalScoreUpdatesRequired` variable this will help us to use one less SLOAD in #L821 and use the stack var there instead.

- [Prime.sol#L821](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L821)


## 2. state variable is already defaulted to zero when defined, there is no need to assign it to zero again

If the old value is equal to the new value, not re-storing the value will avoid a Gsreset (2900 gas)

`nextScoreUpdateRoundId` should not be assigned to 0 here because its already have that value.
- [Prime.sol#L156](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L156)


## 3. Avoid updating storage when the value hasn't changed (ones not mentioned in bot report)

In Solidity, manipulating contract storage comes with significant gas costs. One can optimize gas usage by preventing unnecessary storage updates when the new value is the same as the existing one. If an existing value is the same as the new one, not reassigning it to the storage could potentially save substantial amounts of gas, notably 2900 gas for a 'Gsreset'. This saving may come at the expense of a cold storage load operation ('Gcoldsload'), which costs 2100 gas, or a warm storage access operation ('Gwarmaccess'), which costs 100 gas. Therefore, the gas efficiency of your contract can be significantly improved by adding a check that compares the new value with the current one before any storage update operation. If the values are the same, you can bypass the storage operation, thereby saving gas.

`alphaNumerator`
- [Prime.sol#L243](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L243)

`alphaDenominator`
- [Prime.sol#L244](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L244)

`markets`
- [Prime.sol#L276-L277](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L276-L277)
- [Prime.sol#L295-L298](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L295-L298)
- [Prime.sol#L588](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L588)
- [Prime.sol#L633](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L633)
- [Prime.sol#L733](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L733)

`vTokenForAsset`
- [Prime.sol#L301](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L301)

`revocableLimit`
- [Prime.sol#L322](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L322)

`irrevocableLimit`
- [Prime.sol#L323](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L323)

`unreleasedPSRIncome`
- [Prime.sol#L460](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L460)
- [Prime.sol#L580](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L580)

`unreleasedPLPIncome`
- [Prime.sol#L581](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L581)

`interests`
- [Prime.sol#L629](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L629)
- [Prime.sol#L632](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L632)
- [Prime.sol#L676](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L676)
- [Prime.sol#L677](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L677)
- [Prime.sol#L736](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L736)
- [Prime.sol#L737](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L737)
- [Prime.sol#L786](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L786)
- [Prime.sol#L801](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L801)

`tokens`
- [Prime.sol#L708](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L708)

`totalScoreUpdatesRequired`
- [Prime.sol#L820](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L820)

`pendingScoreUpdates`
- [Prime.sol#L821](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L821)

`tokenAmountAccrued`
- [PrimeLiquidityProvider.sol#L200](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L200)


## 4. `++i` costs less gas than `i++`, especially when it’s used in for-loops (--i/i-- too)

Saves 5 gas per loop

- [Prime.sol#L189](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L189)
- [Prime.sol#L217](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L217)
- [Prime.sol#L225](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L225)
- [Prime.sol#L250](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L250)
- [Prime.sol#L345](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L345)
- [Prime.sol#L355](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L355)
- [Prime.sol#L614](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L614)
- [Prime.sol#L636](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L636)
- [Prime.sol#L711](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L711)
- [Prime.sol#L713](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L713)
- [Prime.sol#L740](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L740)
- [Prime.sol#L766](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L766)
- [Prime.sol#L819](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L819)
- [Prime.sol#L221](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L221)
- [Prime.sol#L745](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L745)
- [Prime.sol#L747](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L747)
- [Prime.sol#L767](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L767)
- [Prime.sol#L828](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L828)
- [Prime.sol#L831](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L831)


## 5. it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied (not in the bot report)

- [Prime.sol#L178](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L178)
- [Prime.sol#L204](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L204)
- [Prime.sol#L211](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L211)
- [Prime.sol#L246](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246)
- [Prime.sol#L335](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L335)
- [Prime.sol#L349](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L349)
- [Prime.sol#L609](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L609)
- [Prime.sol#L625](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L625)
- [Prime.sol#L730](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L730)


## 6. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas. (only the ones that make sense)

- [Prime.sol#L370-L373](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L370-L373)
- [Prime.sol#L421-L424](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L421-L424)
- [Prime.sol#L482-L485](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L482-L485)
- [Prime.sol#L710-L713](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L710-L713)
- [Prime.sol#L744-L747](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L744-L747)
- [Prime.sol#L855-L858](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L855-L858)
- [Prime.sol#L931-L934](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L931-L934)


## 7. before some functions we should check some variables for possible gas save

before transfer we should check for amount being 0 so the function doesnt run when its not gonna do anything

there is a possiblity that `accruedAmount` is 0 because its never checked ahead of this line, which will result in waste of gas in here
- [PrimeLiquidityProvider.sol#L204](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L204)

`amount_` is also never checked to be 0 ahead of the line so its possible that it be 0
- [PrimeLiquidityProvider.sol#L224](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L224)


## 8. array[index] += amount is cheaper than array[index] = array[index] + amount (or related variants)

When updating a value in an array with arithmetic, using array[index] += amount is cheaper than array[index] = array[index] + amount. This is because you avoid an additonal mload when the array is stored in memory, and an sload when the array is stored in storage. This can be applied for any arithmetic operation including +=, -=,/=,*=,^=,&=, %=, <<=,>>=, and >>>=. This optimization can be particularly significant if the pattern occurs during a loop.

Saves 28 gas for a storage array, 38 for a memory array

- [Prime.sol#L588](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L588)
- [Prime.sol#L633](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L633)
- [Prime.sol#L800](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L800)


## 9. Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
you can use this [tool](https://emn178.github.io/solidity-optimize-name/) to get the optimized version for function and properties signatures 


## 10. Use hardcoded address instead address(this)

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's `script.sol` and solmate's `LibRlp.sol` contracts can help achieve this. - [Reference](https://twitter.com/transmissions11/status/1518507047943245824)

- [Prime.sol#L564](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L564)
- [Prime.sol#L682](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L682)
- [Prime.sol#L686](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L686)
- [Prime.sol#L960](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L960)

- [PrimeLiquidityProvider.sol#L217](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L217)
- [PrimeLiquidityProvider.sol#L234](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L234)
- [PrimeLiquidityProvider.sol#L259](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L259)


## 11. Initializers can be marked as payable to save deployment gas

Payable functions cost less gas to execute, because the compiler does not have to add extra checks to ensure that no payment is provided. Initializers can be safely marked as payable, because only the deployer or the factory contract would call the function without carrying any funds. saves 21 gas

- [Prime.sol#L130](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L130)

- [PrimeLiquidityProvider.sol#L90](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L90)
