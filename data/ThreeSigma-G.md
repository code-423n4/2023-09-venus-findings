## Gas Optimizations

[GAS - 1] `PrimeLiquidityProvider` [`accrueTokens(token_)`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249-L272) should return the `accruedAmount` to save gas 
In [lines 570 and 571](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L570-L571) of `Prime`, `accrueTokens` is called and it is followed by another call to the the `PrimeLiquidityProvider` contract to fetch the value of `tokenAmountAccrued[token_]`. By having `accrueTokens` immediately return this value, it would save over 1000 gas on calls to `accrueInterest()` and would improve the `releaseFunds()` function of the `PrimeLiquidityProvider`

In the `releaseFunds()`  function, when the function `accrueTokens` is called in [line 198](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L198-L199) of `PrimeLiquidityProvider`, it is followed again by an update to the `tokenAmountAccrued` mapping to use the new amount of accrued tokens. By immediately returning the `accruedAmount` this extra line would be avoided.

[GAS - 2] `Prime` - The variable [`totalScoreUpdatesRequired`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L93-L94) could be removed
This variable isn't used for anything in the contracts and for off-chain use it can be simply calculated as `totalIrrevocable + totalRevocable`. A view function called `totalScoreUpdatesRequired()` could also be created which returns the  `totalIrrevocable + totalRevocable`, achieving the same result, without having to take a storage slot. Technically it would be off by the tokens that had been minted this round. This variable could also just be emitted in a event triggered in `_startScoreUpdateRound()`

[GAS - 3] `Prime`, part of the function [`xvsUpdated`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L369-L381) could be optimized/simplified for readability and gas.
Instead of 

```solidity
    if (tokens[user].exists && !isAccountEligible) {
        if (tokens[user].isIrrevocable) {
            _accrueInterestAndUpdateScore(user);
        } else {
            _burn(user);
        }
    } else if (!isAccountEligible && !tokens[user].exists && stakedAt[user] > 0) {
        stakedAt[user] = 0;
    } else if (stakedAt[user] == 0 && isAccountEligible && !tokens[user].exists) {
        stakedAt[user] = block.timestamp;
    } else if (tokens[user].exists && isAccountEligible) {
        _accrueInterestAndUpdateScore(user);
    }
```
It could be written as the following, with the same results

```solidity
if (tokens[user].exists) {
        if (tokens[user].isIrrevocable || isAccountEligible) {
            _accrueInterestAndUpdateScore(user);
        } else {
            _burn(user);
        }
    } else if (!isAccountEligible && stakedAt[user] > 0) {
        stakedAt[user] = 0;
    } else if (stakedAt[user] == 0 && isAccountEligible) {
        stakedAt[user] = block.timestamp;
    }
```

[GAS - 4] `Prime` - Function [`isEligible`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L904-L910) could be optimized/simplified for readability and gas
Instead of

```solidity
function isEligible(uint256 amount) internal view returns (bool) {
	if (amount >= MINIMUM_STAKED_XVS) {
		return true;
	} 
		return false;
}
```
It could be written as the following, with the same results

```solidity
function isEligible(uint256 amount) internal view returns (bool) {
    return amount >= MINIMUM_STAKED_XVS);
}
```