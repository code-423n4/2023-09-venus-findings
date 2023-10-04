# [L-01] StakedAt time is not deleted during the issuance of prime tokens
When a directly revocable token is issued, the stakedAt time of the user is deleted  `delete stakedAt[users[i]]`. This is not done when an irrevocable token is issued. The idea behind deleting the user's stakedAt time is to prevent situations where a user already has a staking position. If they decide to withdraw all XVS tokens and only `_accrueInterestAndUpdateScore` is called in the `xvsUpdated` function without clearing the stakedAt time, it could open up the possibility of future minting of prime tokens without staking XVS for the 90-day duration.

## Recommendation
Delete stakedAt time of user durring issuing of irrevocable token.
```diff
                if (userToken.exists && !userToken.isIrrevocable) {
                    _upgrade(users[i]);
                } else {
                    _mint(true, users[i]);
                    _initializeMarkets(users[i]);

+                   delete stakedAt[users[i]];
                }

```

# [L-02] Possible reverting of accrueInterest
The total unreleased income is the amount from the last time `releaseFund` was invoked in the PSR contract. In scenarios where the income variable is `totalIncomeUnreleased == 0` or less than `unreleasedPSRIncome[underlying]`, it is possible for the `uint256 distributionIncome = totalIncomeUnreleased - unreleasedPSRIncome[underlying];` to revert because `unreleasedPSRIncome[underlying]` represents the latest saved income. 

## Recommendation
Add additional check for that:

```solidity
        uint256 distributionIncome;
        if(totalIncomeUnreleased >= unreleasedPSRIncome[underlying]) {
             distributionIncome = totalIncomeUnreleased - unreleasedPSRIncome[underlying];
        }
```

# [NC-01] isEligible function can be optimized
The `isEligible` function can directly return a value instead of using an if condition.

```
    function isEligible(uint256 amount) internal view returns (bool) {
        //@audit GAS: directly return amount >= MINIMUM_STAKED_XVS
        if (amount >= MINIMUM_STAKED_XVS) { 
            return true;
        }

        return false;
    }
```

## Recommendation
Make the following changes:

```solidity
    function isEligible(uint256 amount) internal view returns (bool) {
        return amount >= MINIMUM_STAKED_XVS;
    }
```