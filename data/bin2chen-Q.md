## Findings Summary

| Label | Description |
| - | - |
| [L-01](#l-01-addmarket-needs-to-be-forced-not-to-have-the-same-underlying) | `addMarket()` needs to be forced not to have the same underlying|
| [L-02](#l-02-blocks_per_year-the-block-out-time-in-l2-may-be-modified) | BLOCKS_PER_YEAR The block out time in L2 may be modified|
| [L-03](#l-03-sweeptoken-need-to-limit-remaining-tokens-to-no-less-than-tokenamountaccruedtoken_) | sweepToken() Need to limit remaining tokens to no less than `tokenAmountAccrued[token_]`|



## [L-01] `addMarket()` needs to be forced not to have the same underlying
If different `vToken`s have the same `underlying`, there are multiple problems
1. vTokenForAsset[] will conflict
2. rewards are recorded as `underlying`, and different vTokens will compete with each other for rewards

It is recommended to revert if there is already an `underlying`.
```diff
    function addMarket(address vToken, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
        _checkAccessAllowed("addMarket(address,uint256,uint256)");
        if (markets[vToken].exists) revert MarketAlreadyExists();
+       if (vTokenForAsset[_getUnderlying(vToken)] !=address(0) revert InvalidVToken();         

        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);
        if (!isMarketExist) revert InvalidVToken();

...
    }
```

## [L-02] BLOCKS_PER_YEAR The block out time in L2 may be modified
BLOCK_TIME actually changes every now and then, especially in L2s.For example, the recent Bedrock upgrade in Optimism completely [changed](https://community.optimism.io/docs/developers/bedrock/differences/#the-evm) the block time generation.
Since BLOCKS_PER_YEAR is currently `immutable`, if `BLOCK_TIME` is modified, it will not be able to be adjusted, affecting the calculation of APR.

Suggest to change it to be changeable

## [L-03] sweepToken() Need to limit remaining tokens to no less than `tokenAmountAccrued[token_]`
`sweepToken() ` can be used to move tokens
But there is no restriction if the token moved is a rewarded token
If the remaining token after removal is less than `tokenAmountAccrued[token_]`, it will cause the following 2 methods to underflow
1. `accrueTokens()`
2. `getEffectiveDistributionSpeed()`

suggestion:

```diff
    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
        uint256 balance = token_.balanceOf(address(this));
-       if (amount_ > balance) {
+       if (balance - amount < tokenAmountAccrued[token_]) {    
            revert InsufficientBalance(amount_, balance);
        }
      

        emit SweepToken(address(token_), to_, amount_);

        token_.safeTransfer(to_, amount_);
    }

```

