## Summary
[L-1] No validation for multipliers.

### [L-1] No validation for multipliers.
The `supplyMultiplier` and `borrowMultiplier` for a particular market can be set when adding a new market through the `addMarket` function and can be later updated through the `updateMultipliers` function. Both of these functions have validation if the msg.sender has access. However there is no validation for the data that will be stored in the `supplyMultiplier` and `borrowMultiplier`. If by mistake they are set to 0, this will affect the `_capitalForScore` so that it will always return 0.

#### Recommended Mitigation Steps
Add validation in both `addMarket` and `updateMultipliers` functions.

```solidity
require(supplyMultiplier > 0 && borrowMultiplier > 0, "Multiplier can not be 0.");
```
