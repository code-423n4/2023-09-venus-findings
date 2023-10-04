0. `PrimeLiquidityProvider::sweepToken()` - No `address(0)` check for recipient's `to_` parameter value, risking accidental `safeTransfer()` to zero address.

https://github.com/code-423n4/2023-09-venus/blob/23f5db740d8a794ac563ac32195b675c53042bb4/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L207-L225
https://github.com/code-423n4/2023-09-venus/blob/23f5db740d8a794ac563ac32195b675c53042bb4/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L224

RISK:
Permanent loss of user tokens after botched rescue attempt by `onlyOwner`.

RECOMMENDATION:

```solidity
    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
    ++  if (to_ == address(0)) revert ZeroAddress();
        uint256 balance = token_.balanceOf(address(this));
        if (amount_ > balance) {
            revert InsufficientBalance(amount_, balance);
        }

        emit SweepToken(address(token_), to_, amount_);

        token_.safeTransfer(to_, amount_); 
    }
```

1. Lots of instances where max indexing limit is applied to event declarations which doesnt seem to make much sense.

https://github.com/code-423n4/2023-09-venus/blob/23f5db740d8a794ac563ac32195b675c53042bb4/contracts/Tokens/Prime/Prime.sol#L59-L88

Examples:

```solidity
    /// @notice Emitted when multiplier is updated
    event MultiplierUpdated(
        address indexed market,
        uint256 indexed oldSupplyMultiplier,
        uint256 indexed oldBorrowMultiplier,
        uint256 newSupplyMultiplier,
        uint256 newBorrowMultiplier
    );
    
    /// @notice Emitted when alpha is updated
    event AlphaUpdated(
        uint128 indexed oldNumerator,
        uint128 indexed oldDenominator,
        uint128 indexed newNumerator,
        uint128 newDenominator
    );
    
    
      /// @notice Emitted when mint limits are updated
    event MintLimitsUpdated(
        uint256 indexed oldIrrevocableLimit,
        uint256 indexed oldRevocableLimit,
        uint256 indexed newIrrevocableLimit,
        uint256 newRevocableLimit
    );
    
    /// @notice Emitted when a market is added to prime program
    event MarketAdded(address indexed market, uint256 indexed supplyMultiplier, uint256 indexed borrowMultiplier);
```

RECOMMENDATION:

Only index what is really necessary and possible to index. Not all data types are indexable.
