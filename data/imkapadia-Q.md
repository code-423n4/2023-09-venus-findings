## [L-01] Lack of Multiplier Conversion
**Description:**
In the `addMarket()`, there is a discrepancy between the provided comments and the actual code implementation regarding the conversion of `supplyMultiplier` and `borrowMultiplier` to `1e18`. The comments clearly state that these multipliers should be converted to the `1e18` format. However, there is no code logic performing this conversion. As a result, if these values are meant to be in a different format. This lack of conversion could lead to incorrect interest rate calculations and reward distributions.

**Affected Code:**
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L309