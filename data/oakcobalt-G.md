### Gas 01: Wasteful gas operation in `updateScores()` for-loop
In Prime.sol - `updateScores()`, contains two for-loops - (1) user for-loop (2) market for-loop. The second for-loop can be optimized.

Under current implementation, the market for-loop contains `_executeBoost()` and `_updateScore()` calls for each market and each user. Note that under the hood `_executeBoost()` will always call a more general `accrueInterest(vToken)` function to sync vToken rewards. Since `accrueInterest(vToken)` is not user-specific, this effectively means that for each user the exact same call will be performed even though one call `accrueInterest()` already fully updates `distributionIncome` for a given vToken.

```solidity
//Prime.sol
    function updateScores(address[] memory users) external {
...
   for (uint256 i = 0; i < users.length; ) {
            address user = users[i];
      
            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

            address[] storage _allMarkets = allMarkets;
            for (uint256 j = 0; j < _allMarkets.length; ) {
                address market = _allMarkets[j];
 |>             _executeBoost(user, market);
                _updateScore(user, market);

                unchecked {
                    j++;
                }
            }
...
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L213)
```solidity
//Prime.sol
    function _executeBoost(address user, address vToken) internal {
        if (!markets[vToken].exists || !tokens[user].exists) {
            return;
        }
|>       accrueInterest(vToken);
        interests[vToken][user].accrued += _interestAccrued(vToken, user);
        interests[vToken][user].rewardIndex = markets[vToken].rewardIndex;
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L784)
As seen from above, `accureInterest(vToken)` is nested in (2) for-loops and executed once per each user and each market. This is quite wasteful.

Recommendations:
Consider breaking down `_executeBoost()` and only iterating `accrueInterest(vToken)` once, since every user participates in all markets. See a sample below:
```solidity
//Prime.sol
    function updateScores(address[] memory users) external {
...
|>      bool allMarketsAccrued;
        for (uint256 i = 0; i < users.length; ) {
...
            for (uint256 j = 0; j < _allMarkets.length; ) {
                address market = _allMarkets[j];
|>              if (!allMarketsAccrued) {
                    accrueInterest(market);
                    if (j == _allMarkets.length - 1) {
                        allMarketsAccrued = true;
                    }
                }
|>              if (tokens[user].exists) {
                    interests[market][user].accrued += _interestAccrued(market, user);
                    interests[market][user].rewardIndex = markets[market].rewardIndex;
                }
                _updateScore(user, market);
...
}
```

## Gas 02: Wasteful state update in `_startScoreUpdateRound()`
In `_startScoreUpdateRound()`, any time a new round begins, both `pendingScoreUpdates` and `totalScoreUpdatesRequired` will be rewritten without knowing if `totalScoreUpdatesRequired` changed.

```solidity
//Prime.sol
    function _startScoreUpdateRound() internal {
        nextScoreUpdateRoundId++;
|>      totalScoreUpdatesRequired = totalIrrevocable + totalRevocable;
|>      pendingScoreUpdates = totalScoreUpdatesRequired;
    }
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L818-L821)
When `totalScoreUpdatesRequired` hasn't changed, storage writing twice is a waste of gas. 

Recommendations:
Consider checking whether `totalScoreUpdateRequired` differs from `totalIrrevocable + totalRevocable` first, and only re-write to storage when the value changes.

### Gas 03: Waste of gas to rewrite the '0' value to '0' in Prime.sol - `initialize()`
In Prime.sol - `initialize()`, state variable `nextScoreUpdateRoundId` will be rewritten to `0`. This is wasteful storage writing.

```solidity
//Prime.sol
    function initialize(
...
|>      nextScoreUpdateRoundId = 0;
...
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L156)
```solidity
//PrimeStorage.sol
    /// @notice unique id for next round
    uint256 public nextScoreUpdateRoundId;
```
Recommendations:
Delete `nextScoreUpdateRoundId = 0;`

### Gas 04: `accrueInterest()` wasts gas when `_primeLiquidityProvider` already accured tokens in the same block
In Prime.sol `accrueInterest()`, `accureTokens()` call to `_primeLiquidityProvider` is performed every time, and subsequent math operation of `totalAccruedInPLP - unreleasedPLPIncome[underlying];` will be performed, then subsequent storage writing to `unreleasedPSRIncome[underlying]` and `unreleasedPLPIncome[underlying]` are performed without verifying if both values have changed.

The above math operation plus storage operation are wasteful if (1)`accrueInterest()` is just called in the same block which means `totalAccruedInPLP - unreleasedPLPIncome[underlying];` returns `0` and `0` is added to `distributionIncome` ;(2)OR if either `unreleasedPSRIncome[underlying]` didn't change or `unreleasedPLPIncome[underlying]` didn't change, which resulting in updating storge variable with the same value.

```solidity
//Prime.sol
    function accrueInterest(address vToken) public {
...
|>   _primeLiquidityProvider.accrueTokens(underlying);
        uint256 totalAccruedInPLP = _primeLiquidityProvider.tokenAmountAccrued(underlying);
|>      uint256 unreleasedPLPAccruedInterest = totalAccruedInPLP - unreleasedPLPIncome[underlying];

|>      distributionIncome += unreleasedPLPAccruedInterest;

        if (distributionIncome == 0) {
            return;
        }
|>      unreleasedPSRIncome[underlying] = totalIncomeUnreleased;
|>      unreleasedPLPIncome[underlying] = totalAccruedInPLP;
...
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L554)
```solidity
//PrimeLiquiityProvider.sol
    function accrueTokens(address token_) public {
...
        uint256 blockNumber = getBlockNumber();
        uint256 deltaBlocks = blockNumber - lastAccruedBlock[token_];
        //@audit-info `accrueTokens` will not update when called again in the same block.
        if (deltaBlocks > 0) {
...
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L257)
In addition, `accrueInteret()` is one of the most frequently called functions in many user flows in Prime.sol. For example, it is nested under `_executeBoost()` and most importantly the most gas-intensive function `updateScores()`. This significantly increases the total gas wasted by any function that nests `accrueInterest()` in a for-loop.

Recommendations:
 Consider add check for `unreleasedPLPAccruedInterest`, if `unreleasedPLPAccruedInterest` is '0', we don't add to `distributionIncome` and also don't re-write to `unreleasedPLPIncome[underlying]`;



 