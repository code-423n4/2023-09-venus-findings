### Low 01: Prime.sol will not support market with an underlying token with more than 18 decimals. Capital calculation will revert.
Markets are added at the discretion of selected protocol roles or through voting, however, when a market with an underlying token with more than 18 decimals is created. They will not be supported by current Prime.sol and will cause key score-related functions to revert.

In addition, adding one unsupported market to Prime.sol through `addMarket()` will paralyze the Prime.sol, due to the fact that `_calculateScore()` will revert, which causes key functions such as `_initializeMarkets` to revert, effectively paralyzing Prime.sol contract.

```solidity
//Prime.sol
    function _calculateScore(address market, address user) internal returns (uint256) {
...
        (uint256 capital, , ) = _capitalForScore(xvsBalanceForScore, borrow, supply, market);
|>      capital = capital * (10 ** (18 - vToken.decimals()));
...
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L661)

As seen above, when `decimals()` is greater than 18, a negative exponential will cause EVM to revert.

```solidity
//Prime.sol
    function _initializeMarkets(address account) internal {
        address[] storage _allMarkets = allMarkets;
        for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            accrueInterest(market);

            interests[market][account].rewardIndex = markets[market].rewardIndex;

|>          uint256 score = _calculateScore(market, account);
...
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L631)
Recommendations:
(1) Adding bypass scaling when a token decimal is more than 18 decimals.
(2) Explicitly prohibiting adding market with underlying token with greater than 18 decimals.

### Low 02: No methods to remove or disable a market accounting in Prime.sol, susceptible to a single-point failure
In Prime.sol, all the markets added to Prime.sol will have to be queried in a for-loop through multiple interest or capital-related accounting methods. Since there is no mechanism to remove or disable a single market, if any one of the active markets needs to be halted or excluded for any reason, the entire contract needs to be paused. And the only option to truly remove a problematic market from the loop is a contract upgrade.

Like any defi market, there could be many reasons why a lending/borrowing market needs to be put on pause or even completely removed. The reason can be specific to the issues related to the underlying token protocol. When such a case happens, there is no remove or disable market mechanism in Prime.sol to reduce such impacts. 

```solidity
//Prime.sol
    function addMarket(address vToken, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
...
        markets[vToken].rewardIndex = 0;
        markets[vToken].supplyMultiplier = supplyMultiplier;
        markets[vToken].borrowMultiplier = borrowMultiplier;
        markets[vToken].sumOfMembersScore = 0;
        markets[vToken].exists = true;
|>      allMarkets.push(vToken);
...}
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L303)
As seen from above, a new market is added to the `markets` mapping.

```solidity
//Prime.sol
    function _initializeMarkets(address account) internal {
|>          address[] storage _allMarkets = allMarkets;
        for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            accrueInterest(market);

            interests[market][account].rewardIndex = markets[market].rewardIndex;

            uint256 score = _calculateScore(market, account);
...
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L624)
Important accounting functions will always loop through all the markets. 

Recommendations:
Add a remove market method to allow a single market being removed from the for-loop without impacting the whole contract.

### Low 03: Unnecessary code in Prime.sol `initialize()`
`nextScoreUpdateRoundId` is declared as a state variable. In `initialize()`, it is assign as '0' value which is a redundant state update.
```solidity
//Prime.sol
    function initialize(
        address _xvsVault,
        address _xvsVaultRewardToken,
        uint256 _xvsVaultPoolId,
        uint128 _alphaNumerator,
        uint128 _alphaDenominator,
        address _accessControlManager,
        address _protocolShareReserve,
        address _primeLiquidityProvider,
        address _comptroller,
        address _oracle,
        uint256 _loopsLimit
    ) external virtual initializer {
...
|>      nextScoreUpdateRoundId = 0;
...
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L156)
```solidity
//Prime.sol
    /// @notice unique id for next round
    uint256 public nextScoreUpdateRoundId;
```
Recommendation:
Delete `nextScoreUpdateRoundId = 0;`


### Low 04: Incorrect natspec `@notice` for `lastAccruedBlock`
Natspec `@notice` is important when generating doc for codebase. Current `@notice` for `lastAccruedBlock` is incorrect or misleading. It's currently marked as `the rate at which token is distributed to the Prime contract`, this is not a rate, but the block number of the last accrual block.

```solidity
//PrimeLiquidityProvider.sol
    /// @notice The rate at which token is distributed to the Prime contract
    mapping(address => uint256) public lastAccruedBlock;
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L23)

Recommendation:
Being more precise on documentation increases code readability.

### Low 05: When multiple factors changed between an accrual in Prime.sol, score distribution will be incorrect and unfair

In Prime.sol, whenever an accrual for a user takes place when multiple factors driving a user score changes, the score calculation will always be inaccurate. 

A user score is calculated based on multiple factors such as user xvs and market token balance, market multiplier, and alpha.(see below in `_capitalForScore()` and `calculateScore()`) When more than one of these factors changes, the score calculation will not factor in the change of the factor in the corresponding time period. 

```solidity
//Prime.sol
   function _capitalForScore(
        uint256 xvs,
        uint256 borrow,
        uint256 supply,
        address market
    ) internal view returns (uint256, uint256, uint256) {
...
        uint256 xvsPrice = oracle.getPrice(xvsToken);
|>      uint256 borrowCapUSD = (xvsPrice * ((xvs * markets[market].borrowMultiplier) / EXP_SCALE)) / EXP_SCALE;
|>      uint256 supplyCapUSD = (xvsPrice * ((xvs * markets[market].supplyMultiplier) / EXP_SCALE)) / EXP_SCALE;
...
        if (supplyUSD >= supplyCapUSD) {
            supply = supplyUSD > 0 ? (supply * supplyCapUSD) / supplyUSD : 0;
        }

        if (borrowUSD >= borrowCapUSD) {
            borrow = borrowUSD > 0 ? (borrow * borrowCapUSD) / borrowUSD : 0;
        }
        //@audit-info the return variables are in the underlying token decimals.
        return ((supply + borrow), supply, borrow);
...
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L881-L882)
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
|>          (FixedMath.ln(ratio) * alphaNumerator.toInt256()) / alphaDenominator.toInt256()
        );

        if (lessxvsThanCapital) {
            // capital * e ^ (𝝰 * ln(xvs / capital))
            return FixedMath.uintMul(capital, exponentiation);
        }

        // capital / e ^ (𝝰 * ln(capital / xvs))
        return FixedMath.uintDiv(capital, exponentiation);
...
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/libs/Scores.sol#L59)

Suppose a user's market balance changed to BalanceB from BalanceA at 10000 blocks after a market multiplier updates to MultiplierB from MultiplierA. And the user's capital is affected by MultiplierB. Since the last user score update is before the multiplier update with BalanceA, when the user's reward accrued at 10000 blocks, their old score will be used for a time period that spans both MultiplierB and MultiplierA. However, only MultiplierA will be used to calculate user rewards and this is incorrect and unfair to other users. 

```solidity
//Prime.sol
    function accrueInterestAndUpdateScore(address user, address market) external {
|>       _executeBoost(user, market);
        _updateScore(user, market);
    }
    function _executeBoost(address user, address vToken) internal {
        if (!markets[vToken].exists || !tokens[user].exists) {
            return;
        }

        accrueInterest(vToken);
|>      interests[vToken][user].accrued += _interestAccrued(vToken, user);
        interests[vToken][user].rewardIndex = markets[vToken].rewardIndex;
    }
    function _interestAccrued(address vToken, address user) internal view returns (uint256) {
        uint256 index = markets[vToken].rewardIndex - interests[vToken][user].rewardIndex;
|>       uint256 score = interests[vToken][user].score;
        return (index * score) / EXP_SCALE;
    }
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L920)
As seen from the flow above, the old user score stored in `interests[vToken][user]` will be used to calculate accrual. In the scenario above, the old score is only based on MultiplierA. And the rewardIndex will be a time period spanning two different Multipliers, making the score accrual inaccurate, unfair for other users.

Uneven score distribution will always occur in such scenarios. I consider this low severity due to the fact that `multiplier` or `alpha` updates are protocol global decisions that can be announced to allow users to update their balance before `multiplier` and `alpha` change, and sync their scores after global updates before making further balance change.

Recommendations:
(1) Either as mentioned use off-chain system to advise users against making balance updates that span global state updates;
(2) Or consider updating the accounting structure to include `timestamp` for user score rewritten to properly calculate user score changes.


