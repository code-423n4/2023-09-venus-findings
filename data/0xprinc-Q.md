### 1. Function mismatch in `IPrime.sol` and `Prime.sol`
The function `accrueInterest(address vToken)` is marked as `external` in `IPrime.sol` and `public` in `Prime.sol`. This will lead to an issue when in future the interface is inherited in any other smart contract, then the function `accrueInterest(address vToken)` will not be able to be called internally. <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/IPrime.sol#L9
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L554 <br>
**Mitigation** : either change the visibility of the function to `public` or create another `IPrime.sol` in future to inherit with the required visibility of that function.


### 2. The function `PrimeLiquidityPrivider.sol/setPrimeToken()` is very much vulnerable to frontrun by the function `releaseFunds()` of the same contract.
The prime tokens either releases sends the funds to the old prime contract, or just stop them from going them to the old contract. If the intention is to not let go the funds to the old contract(which is possible due to the frontrun by the `releasefunds` function under certain conditions), then pause the funds transfer using `pauseFundsTransfer()` function and then change the prime contract address.

### 3. The comment in related to the state variable is misleading in `PrimeStorage.sol/STAKING_PERIOD`
This comment says that this variable gives number of days for staking the token, but this actually gives number of seconds for which the tokens should be staked <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L39
<br>
**Mitigation** : change the comment to `number of seconds` or change the variable to `90 days`

### 4. Use of better solidity version.
Solidity version `0.8.13` & `0.8.14` have a security vulnerability related to assembly blocks that write to memory. Almost all in-scope contracts are written in this version, and hence will the upcoming contracts be for the consistency. The issue is fixed in version `0.8.15` and is explained [here](https://soliditylang.org/blog/2022/06/15/solidity-0.8.15-release-announcement/). While the problem is with the legacy optimizer, it is still correct to enforce latest Solidity version (0.8.21) or at least the one enforced in other popular library code as OpenZeppelin (0.8.19).

### 5. Misleading and wrong comment for `PrimeLiquidityProvider.sol/lastAccruedBlock`
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L23-L24    <br>
The comment says 
``` Solidity
    /// @notice The rate at which token is distributed to the Prime contract
```

But the actual use of it is to see the last block at which the `accrueTokens()` function is called. So the better way is 
```solidity
    /// @notice The last block at which the tokens were accrued
```

### 6. Redundant state variable in `PrimeLiquidityProvider.sol`
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L15   <br>
Variable named as `EXP_SCALE = 1e18` is redundant as not used in this contract, its used in `Prime.sol` but is taken from `PrimeStorage.sol` by inheriting that contract.

### 7. Redundant event in `PrimeLiquidityProvider.sol`
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L48 <br>
This event is not used anywhere in this contract.

### 8. These events are not included in the functions they belong in `PrimeLiquidityProvider.sol`
The events `FundsTransferPaused()` and `FundsTransferResumed()` should be emitted when the functions `pauseFundsTransfer()` and `resumeFundsTransfer()` are called. But these are not emitted anytime. 
<br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L51-L54  <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L132-L144 <br>

### 9. better to set the `prime` address in `PrimeLiquidityProvider.sol` during the `initialize` function is called.

### 10. `PrimeLiquidityProvider.sol/setPrimeToken()` lacks input validation fot the prime address to be a contract
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177

### 11. The function `PrimeLiquidityProvider.sol/sweepTokens()` should also include the transfer of some non-erc tokens that don't follow the standard of EIP-20.
Not all the tokens have the methods of `safeTransfer`.
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216


### 12. Event may also include the old and new values of the updating parameter in `Prime.sol`
event name is `UserScoreUpdated()` and contain the old and new score of the user. <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L71

### 13. validation of immutable variables is missing, and this will lead to permanent issue in `Prime.sol/constructor()`
The addresses of `WBNB` and `VBNB` should also be checked to be contract and contain the bytecode. This extra check as the variables are immutable and any mistake will lead to the new contract deploy, and in sometimes the funds to be stuck. <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L107-L108


### 14. An event should be present for the function `Prime.sol/togglePause()` 
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L419 <br>
```solidity
    event isPaused(indexed bool _status);
```


### 15. Function `Prime.sol/claimTimeRemaining()` should check for the eligibility for the user otherwise the eligible user and ineligible users will see the same output at the very first second
At the first second after the `xvsStaked()` is called, the function `claimTimeRemaining()` returns 90 days and for the person who is not eligible, the results are same. This is misleading, and should be changed for the person who is not eligible (meybe revert for them). <br>
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L478