# Venus Prime Gas Report
## [G-01] Prime.addMarket initializes values to zero
### Description
Initializing variables to zero is not necessary since they are default zero; see the inline comments below:
```
    function addMarket(address vToken, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
        _checkAccessAllowed("addMarket(address,uint256,uint256)");
        if (markets[vToken].exists) revert MarketAlreadyExists();

        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);
        if (!isMarketExist) revert InvalidVToken();

        markets[vToken].rewardIndex = 0; //gas, this isn't necessary
        markets[vToken].supplyMultiplier = supplyMultiplier;
        markets[vToken].borrowMultiplier = borrowMultiplier;
        markets[vToken].sumOfMembersScore = 0; //gas, this isn't necessary
        markets[vToken].exists = true;

        vTokenForAsset[_getUnderlying(vToken)] = vToken;

        allMarkets.push(vToken);
        _startScoreUpdateRound();

        _ensureMaxLoops(allMarkets.length);

        emit MarketAdded(vToken, supplyMultiplier, borrowMultiplier);
    }
```
### Recommended Mitigation Steps
```diff
    function addMarket(address vToken, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
        _checkAccessAllowed("addMarket(address,uint256,uint256)");
        if (markets[vToken].exists) revert MarketAlreadyExists();

        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);
        if (!isMarketExist) revert InvalidVToken();

-       markets[vToken].rewardIndex = 0; //gas, this isn't necessary
        markets[vToken].supplyMultiplier = supplyMultiplier;
        markets[vToken].borrowMultiplier = borrowMultiplier;
-       markets[vToken].sumOfMembersScore = 0; //gas, this isn't necessary
        markets[vToken].exists = true;

        vTokenForAsset[_getUnderlying(vToken)] = vToken;

        allMarkets.push(vToken);
        _startScoreUpdateRound();

        _ensureMaxLoops(allMarkets.length);

        emit MarketAdded(vToken, supplyMultiplier, borrowMultiplier);
    }
```

## [G-02] ++variable is cheaper than variable++
### Description
It's 5 gas cheaper to increment/decrement variables like '++variable' compared to 'variable++'. There are many instances in the codebase of the more expensive incrementing format.
### Recommended Mitigation Steps
Use ++variable instead of variable++.
## [G-03] Prime.totalScoreUpdatesRequired should be removed
### Description
The two functions below are the only ones that access `totalScoreUpdatesRequired`:
```
    function _startScoreUpdateRound() internal {
        nextScoreUpdateRoundId++; 
        totalScoreUpdatesRequired = totalIrrevocable + totalRevocable;
        pendingScoreUpdates = totalScoreUpdatesRequired;
    }
    
    function _updateRoundAfterTokenBurned(address user) internal { 
        if (totalScoreUpdatesRequired > 0) totalScoreUpdatesRequired--;

        if (pendingScoreUpdates > 0 && !isScoreUpdated[nextScoreUpdateRoundId][user]) {
            pendingScoreUpdates--;
        }
    }
```
The score updates are fully tracked with the `pendingScoreUpdates` state variable, so the `totalScoreUpdatesRequired` state variable and associated check can be removed from the codebase.
### Recommended Mitigation Steps
Remove `totalScoreUpdatesRequired` since it doesn't do anything.