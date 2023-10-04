### 1. This edge case can also included inside `Scores.sol/calculateScore(...)` function.
This edge case can also be included to save the gas for computing the ratios, exponents and logarithms. <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/Scores.sol#L22-L23
```Solidity
if(alphaNumerator == 0){return capital}
else if(alphaNumerator == alphaDenominator){return xvs}
```

### 2. `PrimeStorage.sol/protocolShareReserve` can be made as immutable as there is no variable that changes this variable in `Prime.sol` after initialize
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L103

Mitigation : make this immutable or make a function that changes the `protocolShareReserve` address variable


### 3. Better to save the variable `PrimeStorage.sol/primeLiquidityProvider` as an instance of `PrimeLiquidityProvider.sol`. 
This variable is been used in the `Prime.sol` always after assigning it the ABI of `PrimeLiquidityProvider.sol` which always needs making a new variable that is used to call the functions of `PrimeLiquidityProvider.sol` externally. <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L559 <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L687 <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L976 <br>


### 4. No need to cache these :
1. balance value in memory in `PrimeLiquidityProvider.sol/sweepTokens()` as it is used only one time.<br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L217

2. variable `blockNumber` as it is used only one time in `PrimeLiquidityProvider.sol/_initializeToken()` <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L288-L298

3. `lastAccruedBlock[token_]` as variable as the variable is used only one time in `PrimeLiquidityProvider.sol/_ensureTokenInitialized()` <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L332

4. `totalStaked` for `_xvsBalanceOfUser()` in `Prime.sol/xvsUpdated()` as it is ussed only one time
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L366



### 5. Better not to initialize uint as `uint i = 0;` and just by `uint i;`
Many instances in `Prime.sol` include <br>
1. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L178
2. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L204
3. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L211
4. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246
5. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L335
6. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L609
7. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L625

### 6. better to cache the variable into memory otherwise the storage is accessed which is expensive
Many instances of in `Prime.sol` include <br>
1. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L178
2. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L211
3. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246
4. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L609
5. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L625

### 7. better to call the function Prime.sol/accrueInterestAndUpdateScore()` instead of increasing the codesize
The function `accrueInterestAndUpdateScore()` has the same lines of code as these so instead of writing these lines in `updateScores()`<br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L213-L214 <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L389 <br>


### 8. Redundant statement 
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L459 <br>
```solidity
        IVToken market = IVToken(vToken);
        unreleasedPSRIncome[_getUnderlying(address(market))] = 0;
```
This statement is redundant as the instance created from address is never used for external call, and is then used to just extract the address itself.


### 9. better to use the `delete` keyword instead of manually setting the variable to default value.
Using `delete` keyword is incentivised by `Ethereum` by returning some gas in computation. So use `delete` here :
```solidity
        delete tokens[user].exists;
        delete tokens[user].isIrrevocable;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L750-L751


### 10. Fetching calldata is better than storage in `Prime.sol/_getUnderlying()` function 
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L932

So, better version of this code will be 
```solidity
        if (vToken == VBNB) {
            return vToken;
```
instead of writing `return VBNB` which is fetched from storage.