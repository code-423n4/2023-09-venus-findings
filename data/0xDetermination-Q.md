# Venus Prime QA Report
## [L-01] `PrimeLiquidityProvider.MAX_DISTRIBUTION_SPEED` may not be appropriate for all tokens
### Impact
The token distribution speed may be set to an inappropriately high value.
### Proof of Concept
`MAX_DISTRIBUTION_SPEED` is a constant equal to `1e18`. Distribution speed units is tokens/block. Because different tokens are valued differently, `1e18` may not be an appropriate max distribution speed for all tokens. The difference here may be especially pronounced for tokens with less decimals like USDC. For example, the max distribution speed for ETH would be 1 ETH per block, or around 1500 USD per block. In contrast, the max distribution speed for USDC could be around 1,000,000,000,000 USD per block since USDC has 6 decimal places on ethereum mainnet/l2s.
### Recommended Mitigation Steps
Consider implementing a variable cap on distribution speed.

## [L-02] Qualifiable supply and borrow calculation is different than the equation in the documentation
### Description
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/README.md#rewards

In the above documentation, qualifiable supply and borrow is different than the pseudocode implementation, which is similar to the actual implementation.

Qualifiable supply and borrow equation:

$\sigma_{i,m} = \min(\tau_i \times borrowMultiplier_m, borrowedAmount_{i,m}) + \min(\tau_i \times supplyMultiplier_m, suppliedAmount_{i,m})$

Pseudocode:
```
borrowUSDCap = toUSD(xvsBalanceOfUser * marketBorrowMultipler)
supplyUSDCap = toUSD(xvsBalanceOfUser * marketSupplyMultipler)
borrowUSD = toUSD(borrowTokens)
supplyUSD = toUSD(supplyTokens)

// borrow side
if (borrowUSD < borrowUSDCap) {
  borrowQVL = borrowTokens
else {
  borrowQVL = borrowTokens * borrowUSDCap/borrowUSD
}

// supply side
if (supplyUSD < supplyUSDCap) {
  supplyQVL = supplyTokens
else {
  supplyQVL = supplyTokens * supplyUSDCap/supplyUSD
}

return borrowQVL + supplyQVL
```

Actual implementation:
```
    function _capitalForScore(
        uint256 xvs,
        uint256 borrow,
        uint256 supply,
        address market
    ) internal view returns (uint256, uint256, uint256) {
        address xvsToken = IXVSVault(xvsVault).xvsAddress();

        uint256 xvsPrice = oracle.getPrice(xvsToken);
        uint256 borrowCapUSD = (xvsPrice * ((xvs * markets[market].borrowMultiplier) / EXP_SCALE)) / EXP_SCALE;
        uint256 supplyCapUSD = (xvsPrice * ((xvs * markets[market].supplyMultiplier) / EXP_SCALE)) / EXP_SCALE;

        uint256 tokenPrice = oracle.getUnderlyingPrice(market);
        uint256 supplyUSD = (tokenPrice * supply) / EXP_SCALE;
        uint256 borrowUSD = (tokenPrice * borrow) / EXP_SCALE;

        if (supplyUSD >= supplyCapUSD) {
            supply = supplyUSD > 0 ? (supply * supplyCapUSD) / supplyUSD : 0;
        }

        if (borrowUSD >= borrowCapUSD) {
            borrow = borrowUSD > 0 ? (borrow * borrowCapUSD) / borrowUSD : 0;
        }

        return ((supply + borrow), supply, borrow);
    }
```
### Recommended Mitigation Steps
The code/docs should match.

## [NC-01] Potential oracle manipulation attack; inflate user score
### Impact
A user could inflate their score and accrue an unfairly large amount of rewards at the expense of other users.

### Proof of Concept
Note that one of the oracles used in the Venus Protocol Oracles repo fetches price from PancakeSwap (https://github.com/VenusProtocol/oracle/blob/develop/contracts/oracles/TwapOracle.sol):
```
/**
 * @title TwapOracle
 * @author Venus
 * @notice This oracle fetches price of assets from PancakeSwap.
 */
contract TwapOracle is AccessControlledV8, TwapInterface {
```
Also note that the supply and borrow components of a user's score calculation depend on the price of the corresponding token:
```
    function _capitalForScore(
        uint256 xvs,
        uint256 borrow,
        uint256 supply,
        address market
    ) internal view returns (uint256, uint256, uint256) {
        address xvsToken = IXVSVault(xvsVault).xvsAddress();

        uint256 xvsPrice = oracle.getPrice(xvsToken);
        uint256 borrowCapUSD = (xvsPrice * ((xvs * markets[market].borrowMultiplier) / EXP_SCALE)) / EXP_SCALE;
        uint256 supplyCapUSD = (xvsPrice * ((xvs * markets[market].supplyMultiplier) / EXP_SCALE)) / EXP_SCALE;

        uint256 tokenPrice = oracle.getUnderlyingPrice(market);
        uint256 supplyUSD = (tokenPrice * supply) / EXP_SCALE;
        uint256 borrowUSD = (tokenPrice * borrow) / EXP_SCALE;

        if (supplyUSD >= supplyCapUSD) {
            supply = supplyUSD > 0 ? (supply * supplyCapUSD) / supplyUSD : 0;
        }

        if (borrowUSD >= borrowCapUSD) {
            borrow = borrowUSD > 0 ? (borrow * borrowCapUSD) / borrowUSD : 0;
        }

        return ((supply + borrow), supply, borrow);
    }
```
If the prices here are from an oracle that gets prices from a single exchange, a flashloan attack could be used to manipulate the prices and manipulate a user's score.
Example:
1. A user's `supplyCapUSD` is larger than the user's `supplyUSD`.
2. The user (who is a contract) takes out a flash loan, makes a trade to inflate the `tokenPrice`, updates their score, then reverses the trade.
3. The user's score is now higher, and the user can accure more rewards at the expense of other users.

This attack is unlikely to happen since the fees incurred from the flashloan attack would likely be larger than the potential gains.
### Recommended Mitigation Steps
Only use decentralized oracles like Chainlink to check price.
## [NC-02] Immutables set in Prime.sol's constructor can be hardcoded to avoid bad inputs
### Description
The `BLOCKS_PER_YEAR`, `WBNB`, and `VBNB` immutables are set to arguments passed to the constructor, so bad input is possible. Since constructor code does not increase deployment costs, it's better to hardcode these values for each chain and pass in the name of the desired chain to the constructor instead.
### Recommended Mitigation Steps
Pass the name of the desired chain to the constructor and use if-else blocks to set the immutable values.
## [NC-03] Prime._executeBoost() naming can be improved
### Description
`_executeBoost()` accrues rewards for users; the term 'boost' is not used anywhere in the documentation. This function can be renamed to improve clarity/readability.
### Recommended Mitigation Steps
Rename '_executeBoost' to '_accrueRewardForUser', or similar.
## [NC-04] Prime._checkAlphaArguments() allows alpha to be anywhere in the range [0, 1]
### Description
It may not be appropriate for the alpha to be able to be set to any value in this range.
### Recommended Mitigation Steps
If desired, add more restrictions on the possible values of alpha.
## [NC-05] Incorrect comment in FixedMath0x.sol
### Description
The math in the below comment is wrong:
```
// For example: log(0.3) = log(e^-1 * e^-0.25 * 1.0471028872385522)
//              = 1 - 0.25 - log(1 + 0.0471028872385522)
```
Additionally, ln() should be used instead of log() for clarity.
### Recommended Mitigation Steps
Change the comment to:
```
// For example: ln(0.3) = ln(e^-1 * e^-0.25 * 1.0471028872385522)
//              = -1 - 0.25 + ln(1 + 0.0471028872385522)
```
## [NC-06] Prime.setLimit() is not called in the initializer
### Description
The protocol plans to issue tokens upon deploying the protocol; limits need to be set in order to issue tokens. Limits are currently not set in the constructor or initializer.
### Recommended Mitigation Steps
Call `setLimit()` in the constructor or initializer.