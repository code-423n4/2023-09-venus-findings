## QA-01. PrimeLiquidityProvider.getEffectiveDistributionSpeed may show incorrect results
## Description
In case if contract has more token balance, then amount that is accrued by rewarders, then `PrimeLiquidityProvider.getEffectiveDistributionSpeed` function [returns `distributionSpeed`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L238).

However, it's possible that `accrueTokens` wasn't called for a long time for a token and as result remaining balance is distributed as well, but still not stored in `tokenAmountAccrued[token_]` variable. As result, function will show that token still accrues rewards, when it's not.
## Recommendation
In order to show correct value, you need to call `accrueTokens` function before. But as this function is not view, that will unlikely be done.