### Advanced Analysis Report for [Venus-Prime](https://github.com/code-423n4/2023-09-venus)

#### Overview
- [Venus-Prime](https://github.com/code-423n4/2023-09-venus) is a complex ecosystem designed for liquidity provision and token distribution, primarily through the [Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol) and [PrimeLiquidityProvider.sol](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol) contracts. 

#### Understanding the Ecosystem:
With attention to key components: 

- [`Prime.sol`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol): Manages the core token logic.
- [`PrimeLiquidityProvider.sol`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol): Manages liquidity and token distribution.
- `AccessControlledV8`(Not in scope but for context): Provides role-based access control.

#### Codebase Quality Analysis:
- The codebase is modular but has high complexity, which increases the risk of errors or vulnerabilities going unnoticed. Below is notes I made during audit for the key contracts [Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol) and [PrimeLiquidityProvider.sol](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol): 

### **[Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol)**:

### 1. Structures and Libraries
- **SafeERC20Upgradeable**: For safe ERC20 interactions.
- **PausableUpgradeable**: For pausing contract functionalities.

### 2. Key State Variables
- `WBNB`: Wrapped BNB address, immutable.
- `VBNB`: Venus BNB address, immutable.
- `BLOCKS_PER_YEAR`: Immutable variable for calculating APR.
- `alphaNumerator` and `alphaDenominator`: For calculating the alpha value.
- `xvsVault`, `xvsVaultRewardToken`, `xvsVaultPoolId`: Related to the XVS vault.
- `protocolShareReserve`, `primeLiquidityProvider`: Addresses for protocol share and liquidity provision.

### 3. Function Modifiers
- `whenNotPaused`: Ensures that the function can only be called when the contract is not paused.

### 4. Event Logging
- `UpdatedAssetsState`: Logs updates to asset states.
- `AlphaUpdated`: Logs updates to the alpha value.
- `MultiplierUpdated`: Logs updates to supply and borrow multipliers.

### 5. Error Handling
- Custom errors like `InvalidCaller`, `InvalidComptroller`, `NoScoreUpdatesRequired`, `MarketAlreadyExists`, `InvalidAddress`, `InvalidBlocksPerYear`, `InvalidAlphaArguments`, `InvalidVToken` are used for better debugging.

### 6. Access Control
- Utilizes `PausableUpgradeable` for pausing functionalities.

### 7. Security Concerns
- Multiple inheritance increases complexity.

### 8. Potential Attack Vectors I noted
- Integer overflow/underflow: Partially mitigated.

### **[PrimeLiquidityProvider.sol](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol)**:

### 1. Structures and Libraries
- **SafeERC20Upgradeable**: For safe ERC20 interactions.
- **AccessControlledV8**: For role-based access control.
- **PausableUpgradeable**: For pausing contract functionalities.

### 2. Key State Variables
- `prime`: Address of the Prime token.
- `tokenDistributionSpeeds`: Mapping for token distribution speeds.
- `lastAccruedBlock`: Mapping for the last block where tokens were accrued.
- `tokenAmountAccrued`: Mapping for the amount of tokens accrued.

### 3. Function Modifiers
- `_checkAccessAllowed`: Custom modifier to check if the caller has the required access.
- `initializer`: Ensures that the function can only be called during contract initialization.
- `onlyOwner`: Restricts certain functions to the contract owner.

### 4. Event Logging
- `TokenDistributionInitialized`: Logs the initialization of token distribution.
- `TokenDistributionSpeedUpdated`: Logs updates to token distribution speeds.
- `PrimeTokenUpdated`: Logs updates to the Prime token address.
- `TokensAccrued`: Logs the amount of tokens accrued.
- `TokenTransferredToPrime`: Logs the transfer of accrued tokens to the Prime contract.

### 5. Error Handling
- Custom errors like `InvalidArguments`, `InvalidDistributionSpeed`, `InvalidCaller`, `TokenAlreadyInitialized`, `InsufficientBalance`, `FundsTransferIsPaused`, `TokenNotInitialized` are used for better debugging.

### 6. Access Control
- Utilizes `AccessControlledV8` for role-based access control.

### 7. Security Concerns
- Multiple inheritance increases complexity.
- Use of unchecked blocks could potentially lead to underflows/overflows.

### 8. Potential Attack Vectors I noted
- Integer overflow/underflow: Partially mitigated.

#### Architecture Recommendations:
- Implement re-entrancy guards for external calls.
- Use OpenZeppelin's `Ownable` for simplified ownership management.
- Implement event logging for critical state changes.

#### Centralization Risks:
- **Possible Risk**: Single point of failure in `prime` contract.
- **Potential Impact**: Loss of funds or unauthorized actions.
- **Mitigation Strategy**: Implement multi-signature requirements for critical functions.

#### Mechanism Review:
- Token distribution is complex and requires accurate setting of distribution speeds.
- Pausing mechanism could be exploited if not properly managed.

#### Systemic Risks:
- Incorrect setting of distribution speeds could lead to imbalances.

#### Areas of Concern:
- Lack of re-entrancy guards.
- Complexity in token distribution logic.

#### Recommendations:
- Implement automated testing for all functions, recommended platforms for this below. 
- Conduct further formal verification and invariant tests.
- Use [Tenderly](https://dashboard.tenderly.co/) and [Defender](defender.openzeppelin.com) for continued monitoring to prevent un-foreseen risks or actions. 

#### Contract Details:
I made function interaction graphs for the key contracts, to better visualize interactions: 

- Here is the [Graph](https://pasteboard.co/FJJdSLcRuyEl.png) for [Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol)
- Here is the [Graph](https://pasteboard.co/jlSM6Wc4xJ4l.png) for [PrimeLiquidityProvider.sol](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol)

#### Conclusion:
- Venus-Prime is a well-structured but complex system. It requires further rigorous testing and auditing to ensure both functionality and security.

### Time spent:
16 hours