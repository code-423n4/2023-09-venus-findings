### Various cases of arbitrary and incorrect score distribution 
The core of Prime.sol is score and rewards accounting for users. There are several factors affecting a user's score or rewards, but based on current implementations they are not handled equally and fairly.

The most truthful way of updating rewards or scores is to strictly match a time period for accrual to the score derived from the state variables(balance, multipliers, alpha, etc.) from that time period. But this is not always implemented strictly, which opens up room for error or exploits at different severity.

- **Case 1: Only balance changed. And balance is not capped.**

This is the most straightforward scenario and is correctly handled. When a user's balance - either XVS or market token changes and their balance is not capped based on their usd value in `_capitalForScore()`, a user score will always be updated at the point of user balance change. The user's old score will be first used to accrue any rewards prior to balance change before writing the new score to storage.

- **Case 2: Only balance changed. But balance is capped.**

This is not currently handled. When a user's balance is capped, current implementation in `_captialForScore()` is converting both XVS and market underlying token values into USD values, although seems rational this introduces a new dynamic factor to score calculation that is almost impossible to track - token price change. 

```solidity
//Prime.sol
    function _capitalForScore(
        uint256 xvs,
        uint256 borrow,
        uint256 supply,
        address market
    ) internal view returns (uint256, uint256, uint256) {
...
        uint256 tokenPrice = oracle.getUnderlyingPrice(market);
        uint256 supplyUSD = (tokenPrice * supply) / EXP_SCALE;
        uint256 borrowUSD = (tokenPrice * borrow) / EXP_SCALE;
        if (supplyUSD >= supplyCapUSD) {
|>          supply = supplyUSD > 0 ? (supply * supplyCapUSD) / supplyUSD : 0;
        }
        if (borrowUSD >= borrowCapUSD) {
|>          borrow = borrowUSD > 0 ? (borrow * borrowCapUSD) / borrowUSD : 0;
        }
        return ((supply + borrow), supply, borrow);
...
```

A token price can change at any time, which results in a user's capped `capital` token amount fluctuate with token price, Or a user who is not previously capped became caped due to price change and now their score will vary. 

This creates a situation where the score in storage will be inaccurate relative to the actual token price at the time and the rewards update for a user will create an incorrect and arbitrary score distribution - some users' scores are updated based on the new token price, and some others may not. 

Furthermore, this allows malicious users to exploit the incorrect score distribution and game the rule. 
When some users can target other users to either change score distribution or decide not to change the score when it's to their benefit, and repeatedly take advantage of price change opportunities, the reward system is rigged in that it allows opportunistic gains at the expense of others. See details in my HM submission.

Recommendations:
This issue can be resolved by changing the denomination of the cap from USD value to token amounts in `_captialForScore()`, which eliminates the token price factor altogether.

- **Case 3: Only `alpha` or `multiplier` changed.**

This will require score revisions for every user due to `alpha` and `multiplier` are baked into score calculations as seen in `_capitalForScore()`(Prime.sol) and `calculateScore()`(Scores.sol). 

```solidity
//Scores.sol
    function calculateScore(
        uint256 xvs,
        uint256 capital,
        uint256 alphaNumerator,
        uint256 alphaDenominator
    ) internal pure returns (uint256) {
...
        // e ^ ( ln(ratio) * 𝝰 )
        int256 exponentiation = FixedMath.exp(
 |>         (FixedMath.ln(ratio) * alphaNumerator.toInt256()) / alphaDenominator.toInt256()
        );
        if (lessxvsThanCapital) {
            // capital * e ^ (𝝰 * ln(xvs / capital))
            return FixedMath.uintMul(capital, exponentiation);
        }
        // capital / e ^ (𝝰 * ln(capital / xvs))
        return FixedMath.uintDiv(capital, exponentiation);
...
```

However, the current implementation is (1) vulnerable to attacks (2) current mitigation logics (protocols' script to mass update users scores through `updateScores()` (see [docs](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/README.md#update-cap-multipliers-and-alpha)) is faulty and gas intensive (3) Only mitigate but not resolve the issue.

See details on (1) and (2) in my HM and GAS submissions. Briefly, `updateScores()` is vulnerable to an infinite-loop condition which can be easily exploited by a malicious actor to disable protocol safeguards.

Even if the vulnerability in (1) & (2) are fixed. The protocol is still dealing with (3) which means the protocol will always have to pay for a lot of gas to sync every user score update to date, which will be an increasingly difficult process as the Prime.sol gains more traction. In addition, the uneven and incorrect score distribution will still exist for some period of time.

Recommendations:
Apart from the fixes proposed in (1) & (2), one may argue it may be worthwhile to consider updating the score accounting structure to account for user score updated round or timestamp

This means that (1) each time a user score is updated a timestamp or block number will be recorded. (2) Rewards accumulation over any given period of time is linear such that total rewards can be scaled over different lengths of time. 

Because of the new state write, this for sure will increase individual gas costs for single-user score updates, but the goal is to eliminate protocol mass score updates which are intensive and increasingly challenging over time. In addition, (2) will only be required when an accrual spans over two rounds which is not a frequent occurrence, and again the extra gas cost will be shared among users, not the protocol which is more manageable for everyone.













 









### Time spent:
30 hours