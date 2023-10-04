# Low

### Prime.sol: Users with issued tokens can instantly re-enter the protocol if they were in the 90 days waiting period

If an user is issued a prime token while waiting for the claim period, his `stakedAt` is not zeroed. If the user burns this token in the future by withdrawing from the XVS vault, he can earn it back by calling the `claim()` function, bypassing the 90 day waiting period.

In order to fix this, consider setting the `stakedAt[user]` to zero in the token issue, as shown below:

```
                } else {
                    _mint(true, users[i]);
                    _initializeMarkets(users[i]);
                    stakedAt[user] = 0;
                }
```

### Prime.sol: In case of a prime contract migration, earned yield cannot be claimed if the previous contract's balance + the PSR funds are not enough to cover them

In `_claimInterest()`, the PLP unreleased funds are transfered in case the prime contract and the PSR unreleased funds are not enough to cover the interest claim. If there's a prime contract migration, this call will revert as the previous prime contract will not be authorized to claim the PLP tokens. This migration will cause users who have earned interest to only be able to claim what's in the contract and the PSR, if such PSR has not migrated to a new prime token yet. Worst case scenario, the PSR has migrated as well and only the previous prime contract balance can cover the users claims.

Submitting this as low as it's possible for the developers to remigrate to the previous contract in order to fix this. Developers can also claim the interest in the users stead before migrating.

### PrimeLiquidityProvider.sol: Governance shouldn't sweep XVS tokens

If this happens, `getEffectiveDistributionSpeed()` could potentially have a balance lower than accrued value, which leads to this function always reverting whenever it's called.

# Non-Critical

### Prime.sol: Remove comptroller check

`updateAssetsState()` requires a redudant comptroller argument, given there's only one comptroller and the check serves no purpose in the function. 

### PrimeLiquidityProvider.sol: Max distribution speed could be too low in case of future tokens with very high total supply

Tokens, like SHIBA for example, have an extremely high total supply which leads to a very low USD value per token. Since the maximum distribution speed is 1e18 per block, if the protocol includes tokens like these in the future, the reward speed could be too insignificant.

### Scores.sol: Improved math

In line 58, when calculating the exponentiation, consider changing e ^ ( ln(ratio) * 𝝰 ) to ratio^alpha instead, since they're mathematically equivalent. 