## [G-01] Events are not indexed

The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to filter the events efficiently.

Recommend adding the `indexed` keyword in each event,

```
    event PrimeTokenUpdated(address oldPrimeToken, address newPrimeToken);
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L36C1-L36C75

```
    /// @notice Emitted when funds transfer is paused
    event FundsTransferPaused();

    /// @notice Emitted when funds transfer is resumed
    event FundsTransferResumed();

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L50C1-L55C1

## [G-02] Move storage pointer to top of function to avoid offset calculation
We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.

```
 function updateScores(address[] memory users) external {
        if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
        if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

        for (uint256 i = 0; i < users.length; ) {
            address user = users[i];

            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

            address[] storage _allMarkets = allMarkets;
            for (uint256 j = 0; j < _allMarkets.length; ) {
                address market = _allMarkets[j];
                _executeBoost(user, market);
                _updateScore(user, market);

                unchecked {
                    j++;
                }
            }

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200C4-L220C1

```
  function issue(bool isIrrevocable, address[] calldata users) external {
        _checkAccessAllowed("issue(bool,address[])");

        if (isIrrevocable) {
            for (uint256 i = 0; i < users.length; ) {
                Token storage userToken = tokens[users[i]];
                if (userToken.exists && !userToken.isIrrevocable) {
                    _upgrade(users[i]);
                } else {
                    _mint(true, users[i]);
                    _initializeMarkets(users[i]);
                }
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331C3-L342C18

```
 function _burn(address user) internal {
        if (!tokens[user].exists) revert UserHasNoPrimeToken();

        address[] storage _allMarkets = allMarkets;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L725C4-L729C1

## [G-03] Cache state variables outside of loop to avoid reading/writing storage on every iteration
Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a Gwarmaccess (100 gas) per loop iteration.

```
 for (uint256 i; i < numTokens; ) {
            _initializeToken(tokens_[i]);
            _setTokenDistributionSpeed(tokens_[i], distributionSpeeds_[i]);
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L103C8-L105C76



## [G-04] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

```
 if (amount > asset.balanceOf(address(this))) {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L682C8-L682C55

```
    if (amount > asset.balanceOf(address(this))) {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L686C9-L686C59

```
    IProtocolShareReserve(protocolShareReserve).getPercentageDistribution(
                address(this),
                IProtocolShareReserve.Schema.SPREAD_PRIME_CORE
            );
			
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L959C9-L962C15

```
        uint256 balance = token_.balanceOf(address(this));
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L217C1-L217C59

```
        uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L234

```
   uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L259C10-L259C82

## [G-05] State variables only set in the constructor should be declared immutable
The solidity compiler will directly embed the values of immutable variables into your contract bytecode and therefore will save you from incurring a Gsset (20000 gas) when you set storage variables in the constructor; a Gcoldsload (2100 gas) when you access storage variables for the first time in a transaction, and a Gwarmaccess (100 gas) for each subsequent access to that storage slot.

```
 constructor(address _wbnb, address _vbnb, uint256 _blocksPerYear) {
        if (_wbnb == address(0)) revert InvalidAddress();
        if (_vbnb == address(0)) revert InvalidAddress();
        if (_blocksPerYear == 0) revert InvalidBlocksPerYear();
        WBNB = _wbnb;
        VBNB = _vbnb;
        BLOCKS_PER_YEAR = _blocksPerYear;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L103C4-L109C42



## [G-06] Using storage instead of memory for structs/arrays saves gas
Using a memory pointer for a storage struct/array will effectively load all the fields of that data type from storage (SLOAD) into memory (MSTORE). Using a storage pointer will allow you to read specific fields from storage as you need them. If you are not going to use all of the fields of your data type then you should use a storage pointer so that you don’t incur extra Gcoldsload (2100 gas) for fields that you will never use.

```
function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
        address[] storage _allMarkets = allMarkets;
        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174C5-L176C95

```
 if (amount > asset.balanceOf(address(this))) {
            address[] memory assets = new address[](1);
            assets[0] = address(asset);
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L682C8-L684C40


## [G-07] Arithmatic operations can be unchecked, since there is no underflow or overflow

```
  uint256 supply = (exchangeRate * balanceOfAccount) / EXP_SCALE;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L501C7-L501C72

```
 uint256 supply = (exchangeRate * balanceOfAccount) / EXP_SCALE;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L654C8-L654C72

```
  uint256 index = markets[vToken].rewardIndex - interests[vToken][user].rewardIndex;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L919C7-L919C91


```
  uint256 incomePerBlockForDistributionFromMarket = (totalIncomePerBlockFromMarket * _distributionPercentage())
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L972C7-L972C118

```
  uint256 userYearlyIncome = (userScore * _incomeDistributionYearly(vToken)) / totalScore;
        uint256 totalCappedValue = totalCappedSupply + totalCappedBorrow;

        if (totalCappedValue == 0) return (0, 0);

        uint256 userSupplyIncomeYearly = (userYearlyIncome * totalCappedSupply) / totalCappedValue;
        uint256 userBorrowIncomeYearly = (userYearlyIncome * totalCappedBorrow) / totalCappedValue;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L1004C7-L1011C1

```
      uint256 accruedSinceUpdate = deltaBlocks * distributionSpeed;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L263C11-L263C78

## [G-08] += Costs More Gas Than =+ For State Variables

```
 amount += BLOCKS_PER_YEAR * totalIncomePerBlockFromPLP;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L978C8-L978C64

```
  tokenAmountAccrued[token_] += tokenAccrued;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L266C15-L266C60

## [G-09] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```
 function accrueInterest(address vToken) public {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L554C4-L554C53

```
    function getInterestAccrued(address vToken, address user) public returns (uint256) {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L597C1-L597C89

```
  function accrueTokens(address token_) public {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249C3-L249C51

## [G-10] Nested if is cheaper than single statement

```
if (tokens[user].exists && !isAccountEligible) {
            if (tokens[user].isIrrevocable) {
                _accrueInterestAndUpdateScore(user);
            } else {
                _burn(user);
            }
        } else if (!isAccountEligible && !tokens[user].exists && stakedAt[user] > 0) {
            stakedAt[user] = 0;
        } else if (stakedAt[user] == 0 && isAccountEligible && !tokens[user].exists) {
            stakedAt[user] = block.timestamp;
        } else if (tokens[user].exists && isAccountEligible) {
            _accrueInterestAndUpdateScore(user);
        }
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L369C9-L381C10

```
    if (distributionSpeed > 0 && balanceDiff > 0) {
                uint256 accruedSinceUpdate = deltaBlocks * distributionSpeed;
                uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L262C9-L265C1

## [G-11] Caching a variable that is used once just wastes Gas

```
  function _mint(bool isIrrevocable, address user) internal {
        if (tokens[user].exists) revert IneligibleToClaim();

        tokens[user].exists = true;
        tokens[user].isIrrevocable = isIrrevocable;

        if (isIrrevocable) {
            totalIrrevocable++;
        } else {
            totalRevocable++;
        }

        if (totalIrrevocable > irrevocableLimit || totalRevocable > revocableLimit) revert InvalidLimit();

        emit Mint(user, isIrrevocable);
```


https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L704C3-L718C40


## [G-12] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

```
    if (distributionSpeed > 0 && balanceDiff > 0) {
                uint256 accruedSinceUpdate = deltaBlocks * distributionSpeed;
                uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);

                tokenAmountAccrued[token_] += tokenAccrued;
                emit TokensAccrued(token_, tokenAccrued);
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L262C9-L267C58