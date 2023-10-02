### Low 01: Prime.sol will not support market with an underlying token with more than 18 decimals. Capital calculation will revert.
Markets are added at the discretion of selected protocol roles, however, when a market with an underlying token with more than 18 decimals is created. They will not be supported by current Prime.sol and will cause key score-related functions to revert.

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
```solidity
//Prime.sol
    /// @notice unique id for next round
    uint256 public nextScoreUpdateRoundId;
```
Recommendation:
Delete `nextScoreUpdateRoundId = 0;`

### Low 04: `alpha` update might cause uneven or unpredictable score distribution for some period of time
`alpha` is not likely to be changed based on doc -`A default weight of 0.5 weight has been evaluated as a good ratio and is not likely to be changed. A higher value will only be needed if we want to attract more XVS stake from the Prime token holders at the expense of supply/ borrow rewards`. 

However, when `alpha` is changed on a rare occasion, the process where all the user scores are synced with the new `alpha` value is unpredictable to potentially lead to an uneven score distribution where some user updated their score with the new `alpha` value, some others` score is not updated as fast. The ratio of score distribution will be temporarily untruthful and depend on a more or less organic user process. 

I think this is low severity due to (1) the fact that Prime.sol currently implements `isScoreUpdated` mapping to keep track of all user scores can be synced if a new round has started; (2) `alpha` change is a predictable event with a predetermined time and value, such that ample announce can be made to ensure all users are prepared. 

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
            (FixedMath.ln(ratio) * alphaNumerator.toInt256()) / alphaDenominator.toInt256()
        );

        if (lessxvsThanCapital) {
            // capital * e ^ (𝝰 * ln(xvs / capital))
            return FixedMath.uintMul(capital, exponentiation);
        }

        // capital / e ^ (𝝰 * ln(capital / xvs))
        return FixedMath.uintDiv(capital, exponentiation);
...
```
(https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/libs/Scores.sol#L57-L68)

As seen in score calculation, `alpha` value is a universal weight baked into user score.

Recommendations:
Since `alpha` update is a rare occurrence and affects every user, off-chain methods could be used to alert users of pending changes of `alpha` ahead of time to encourage users to update their score in time. Protocol can also assist in the process by mass updating the user's score to sync with the new `alpha`.

 


### NC 01: Incorrect natspec `@notice` for `lastAccruedBlock`
Natspec `@notice` is important when generating doc for codebase. Current `@notice` for `lastAccruedBlock` is incorrect. It's currently marked as `the rate at which token is distributed to the Prime contract`, this is not a rate, but the block number of the last accrual block.

```solidity
//PrimeLiquidityProvider.sol
    /// @notice The rate at which token is distributed to the Prime contract
    mapping(address => uint256) public lastAccruedBlock;
```


Recommendation:
Being more precise on documentation increases readability.




