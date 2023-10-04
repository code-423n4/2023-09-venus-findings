## Summary

### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [G&#x2011;01] | block.number can be directly used instead of creating separate function | 1 |
| [G&#x2011;02] | `whenNotPaused` modifier can be used instead of creating condition | 1 |
| [G&#x2011;03] | Use assembly to emit events | 10 |
| [G&#x2011;04] | Use assembly to check for address(0) | 9 |

### [G&#x2011;01]  block.number can be directly used instead of creating separate function
block.number is a global variable that can be used directly. Avoid the separate function to fetch the block.number value and save gas. Removing extra bytes from contract will also save some deployment cost.

There is 1 instance of this issue which can be checked [here](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249-L301)

```diff
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

    function accrueTokens(address token_) public {
        _ensureZeroAddress(token_);

        _ensureTokenInitialized(token_);

-        uint256 blockNumber = getBlockNumber();
-        uint256 deltaBlocks = blockNumber - lastAccruedBlock[token_];
+        uint256 deltaBlocks = block.number - lastAccruedBlock[token_];

        if (deltaBlocks > 0) {
            uint256 distributionSpeed = tokenDistributionSpeeds[token_];
            uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

            uint256 balanceDiff = balance - tokenAmountAccrued[token_];
            if (distributionSpeed > 0 && balanceDiff > 0) {
                uint256 accruedSinceUpdate = deltaBlocks * distributionSpeed;
                uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);

                tokenAmountAccrued[token_] += tokenAccrued;
                emit TokensAccrued(token_, tokenAccrued);
            }

-            lastAccruedBlock[token_] = blockNumber;
+            lastAccruedBlock[token_] = block.number;
        }
    }

-    /// @notice Get the latest block number
-    /// @return blockNumber returns the block number
-    function getBlockNumber() public view virtual returns (uint256) {
-        return block.number;
-    }

    /**
     * @notice Initialize the distribution of the token
     * @param token_ Address of the token to be intialized
     * @custom:event Emits TokenDistributionInitialized event
     * @custom:error Throw TokenAlreadyInitialized if token is already initialized
     */
    function _initializeToken(address token_) internal {
        _ensureZeroAddress(token_);
-        uint256 blockNumber = getBlockNumber();
        uint256 initializedBlock = lastAccruedBlock[token_];

        if (initializedBlock > 0) {
            revert TokenAlreadyInitialized(token_);
        }

        /*
         * Update token state block number
         */
-        lastAccruedBlock[token_] = blockNumber;
+        lastAccruedBlock[token_] = block.number;

        emit TokenDistributionInitialized(token_);
    }
```

### [G&#x2011;02]  `whenNotPaused` modifier can be used instead of creating condition
In `PrimeLiquidityProvider.sol`, instead of creating not paused condition, a `whenNotPaused` modifier can be used. The contract already inherits the `PausableUpgradeable.sol`. This will save gas by removing extra bytes.

There is 1 instance of this issue which can checked [here](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L194-L195)

```diff
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

-    function releaseFunds(address token_) external {
+    function releaseFunds(address token_) external whenNotPaused {
        if (msg.sender != prime) revert InvalidCaller();
-        if (paused()) {
-            revert FundsTransferIsPaused();
-        }

        accrueTokens(token_);
        uint256 accruedAmount = tokenAmountAccrued[token_];
        tokenAmountAccrued[token_] = 0;

        emit TokenTransferredToPrime(token_, accruedAmount);

        IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);
    }
```

### [G&#x2011;03]  Use assembly to emit events
Assembly can be used to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs.
Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

There are 10 instances of this issue which can be checked [here](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L51-L94)

### [G&#x2011;04]  Use assembly to check for address(0)
Use assembly to check for zero address saves 6 gas per instance.

There are 9 instances of this which can be checked below,

[instance 1-2](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L104-L105) 

[instance 3-8](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L143-L148)

[instance 9](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L457)