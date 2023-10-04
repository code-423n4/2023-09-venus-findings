[G-01] No need of variable that are used only once
**Description:**
In the `PrimeLiquidityProvidor` contract, there are local variables that are declared but only accessed once within the function. These variables serve no purpose as they are only utilized once within the specific function where they are declared. Declaring such variables in the function is redundant and results in unnecessary gas consumption during function execution.

**Affected Code:**
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L233
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L234
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L235
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L288