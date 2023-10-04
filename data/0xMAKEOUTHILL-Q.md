1. [ **Low** ] - `setTokensDistributionSpeed` can be temporary DOS-ed

 In **PrimeLiquidityProvider.sol** whenever a token is swept by calling `sweepToken`

```
function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
        uint256 balance = token_.balanceOf(address(this));
        if (amount_ > balance) {
            revert InsufficientBalance(amount_, balance);
        }

        emit SweepToken(address(token_), to_, amount_);

        token_.safeTransfer(to_, amount_);
    }
```

the accrued amount for that token is not removed `tokenAmountAccrued[token_]` (sponsor stated that `sweepToken` will be invoked by a VIP (Venus Improvement Proposal) for ANY token in the contract). However that's not actually the problem. Whenever a privileged user wants to `setTokensDistributionSpeed` for an already **swept** token, it will be temporary unavailable because the function will revert.

```
function setTokensDistributionSpeed(address[] calldata tokens_, uint256[] calldata distributionSpeeds_) external {
        _checkAccessAllowed("setTokensDistributionSpeed(address[],uint256[])");
        uint256 numTokens = tokens_.length;

        if (numTokens != distributionSpeeds_.length) {
            revert InvalidArguments();
        }

        for (uint256 i; i < numTokens; ) {
            _ensureTokenInitialized(tokens_[i]);
            _setTokenDistributionSpeed(tokens_[i], distributionSpeeds_[i]);

            unchecked {
                ++i;
            }
        }
    }
```

Nothing fancy here, just some checks. Let's go into `_setTokenDistributionSpeed`.

```
function _setTokenDistributionSpeed(address token_, uint256 distributionSpeed_) internal {
        if (distributionSpeed_ > MAX_DISTRIBUTION_SPEED) {
            revert InvalidDistributionSpeed(distributionSpeed_, MAX_DISTRIBUTION_SPEED);
        }

        if (tokenDistributionSpeeds[token_] != distributionSpeed_) {
            // Distribution speed updated so let's update distribution state to ensure that
            //  1. Token accrued properly for the old speed, and
            //  2. Token accrued at the new speed starts after this block.
            accrueTokens(token_);

            // Update speed
            tokenDistributionSpeeds[token_] = distributionSpeed_;

            emit TokenDistributionSpeedUpdated(token_, distributionSpeed_);
        }
    }
```
So now we want to **decrease** or **increase** the distribution speed, for that to happen it must first go through `accrueTokens`.

```
function accrueTokens(address token_) public {
        _ensureZeroAddress(token_);

        _ensureTokenInitialized(token_);

        uint256 blockNumber = getBlockNumber();
        uint256 deltaBlocks = blockNumber - lastAccruedBlock[token_];

        if (deltaBlocks > 0) {
            uint256 distributionSpeed = tokenDistributionSpeeds[token_];
            uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

            //!!!!!! ATTENTION HERE
            uint256 balanceDiff = balance - tokenAmountAccrued[token_];

            if (distributionSpeed > 0 && balanceDiff > 0) {
                uint256 accruedSinceUpdate = deltaBlocks * distributionSpeed;
                uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);

                tokenAmountAccrued[token_] += tokenAccrued;
                emit TokensAccrued(token_, tokenAccrued);
            }

            lastAccruedBlock[token_] = blockNumber;
        }
    }
```

As you can see here `uint256 balanceDiff = balance - tokenAmountAccrued[token_];`. The `balance` of the recently **swept** token can be lower than the accrued amount. So that function will always revert if `balance` < `tokenAmountAccrued[token_]` which will further interfere with changing the distribution speed of the token.