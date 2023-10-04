1) In the Prime.sol, getPendingInterests() loads allMarkets into a storage variable, but does not update allMarkets variable. As the intention was readonly, it is better to read the variable in memory for cost saving.

```
function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
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

        return pendingInterests;
    }
```
