## QA Report

### Low Risk Issues

| |Issue|
|-|:-|
| [L-01] | `getPendingInterests` will always revert if market contain VBNB |  
| [L-02] | `updateMultipliers`, `updateAlpha` and `addMarket` can be called even `pendingScoreUpdates` not 0 | 
| [L-03] | `sweepToken` can break `tokenAmountAccrued` accounting  | 


## Low Risk Issues


### [L-01] `getPendingInterests` will always revert if market contain VBNB
Calling `getPendingInterests` will always revert if market contain VBNB because VBNB don't have `underlying()` read functions.

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L194

```solidity
    function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
        address[] storage _allMarkets = allMarkets;
        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

        for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            uint256 interestAccrued = getInterestAccrued(market, user);
            uint256 accrued = interests[market][user].accrued;

            pendingInterests[i] = PendingInterest({
@>>             market: IVToken(market).underlying(),
                amount: interestAccrued + accrued
            });

            unchecked {
                i++;
            }
        }

        return pendingInterests;
    }
```

Need to be updated using `_getUnderlying` so it will work for all VToken including VBNB.

```diff
    function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
        address[] storage _allMarkets = allMarkets;
        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

        for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            uint256 interestAccrued = getInterestAccrued(market, user);
            uint256 accrued = interests[market][user].accrued;

            pendingInterests[i] = PendingInterest({
-                market: IVToken(market).underlying(),
+                market: _getUnderlying(market),
                amount: interestAccrued + accrued
            });

            unchecked {
                i++;
            }
        }

        return pendingInterests;
    }
```

### [L-02] `updateMultipliers`, `updateAlpha` and `addMarket` can be called even `pendingScoreUpdates` not 0
The mentioned functions can still be called even when inside score update period (`pendingScoreUpdates` not 0). Check need to be performed to finish previous `pendingScoreUpdates` to avoid unexpected protocol state.

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L237-L255
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L263-L280
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L288-L309

```diff
    function addMarket(address vToken, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
        _checkAccessAllowed("addMarket(address,uint256,uint256)");
        if (markets[vToken].exists) revert MarketAlreadyExists();
+       if (pendingScoreUpdates > 0) revert InsideUpdatePeriod();

        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);
        if (!isMarketExist) revert InvalidVToken();

        markets[vToken].rewardIndex = 0;
        markets[vToken].supplyMultiplier = supplyMultiplier;
        markets[vToken].borrowMultiplier = borrowMultiplier;
        markets[vToken].sumOfMembersScore = 0;
        markets[vToken].exists = true;

        vTokenForAsset[_getUnderlying(vToken)] = vToken;

        allMarkets.push(vToken);
        _startScoreUpdateRound();

        _ensureMaxLoops(allMarkets.length);

        emit MarketAdded(vToken, supplyMultiplier, borrowMultiplier);
    }
```

### [L-03] `sweepToken` can break `tokenAmountAccrued` accounting
When `sweepToken` is called, it can withdraw reward token, but it doesn't check and decrease `tokenAmountAccrued`. Could potentially break the accounting and cause issue. Consider to check and update `tokenAmountAccrued` if necessary. 

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225

```diff
    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
        uint256 balance = token_.balanceOf(address(this));
        if (amount_ > balance) {
            revert InsufficientBalance(amount_, balance);
        }
       
+        if (tokenAmountAccrued[token_] > 0) {
+          tokenAmountAccrued[token_] = tokenAmountAccrued[token_] - amount_;
+        }
        
        emit SweepToken(address(token_), to_, amount_);

        token_.safeTransfer(to_, amount_);
    }
```