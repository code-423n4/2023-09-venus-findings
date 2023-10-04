## Gas Optimizations

## Sumary

| No|Issue|Instances|
|-|:-|:-:|
|[G-01]|Gas saving is achieved by removing the delete keyword|1|
|[G-02]|Use a hardcoded address instead of address(this)|7|
|[G-03]|Use uint256(1)/uint256(2) instead for true and false boolean sta|10|
|[G-04]|Make 3 event parameters indexed when possible|4 |
|[G-05]|Do not calculate constants|1|
|[G-06]|Amounts should be checked for 0 before calling a transfer|4|
|[G-07]|Loop best practice to save gas|4|
|[G-08]|Unnecessary look up in if condition|4|
|[G-09]|+= costs more gas than = + for state variables|7|
|[G-10]|Counting down in for statements is more gas efficient|12
|[G-11]|Use do while loops instead of for loops|12
|[G-12]|Avoid emitting constants.|16|
|[G-13]|Use selfbalance() instead of address(this).balance|7|
|[G-14]|Non efficient zero initialization|12|
|[G-15]|Don't initialize variables with default value|8
|[G-16]|Pre-increments and pre-decrements are cheaper than post-increments and post-decrements|15|
|[G-17]|Not using the named return variable when a function returns, wastes deployment gas|5|
|[G-18]|Using storage instead of memory for structs/arrays saves gas|4|
|[G-19]|Public Functions To External|4|

## [G-1] Gas saving is achieved by removing the delete keyword (~60k)
30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity

fike: Tokens/Prime/Prime.sol

352  delete stakedAt[users[i]]
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L352

## [G-2] Use a hardcoded address instead of address(this)
It can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract’s address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

```solidity

file: Tokens/Prime/Prime.sol

541   _reentrancyStatus = true(protocolShareReserve).getUnreleasedFunds(
            comptroller,
            IProtocolShareReserve.Schema.SPREAD_PRIME_CORE,
            address(this),

682     if (amount > asset.balanceOf(address(this)))  

586  if (amount > asset.balanceOf(address(this))) {
                IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
                unreleasedPLPIncome[underlying] = 0;
            }
957    function _distributionPercentage() internal view returns (uint256) {
        return
            IProtocolShareReserve(protocolShareReserve).getPercentageDistribution(
                address(this),
                IProtocolShareReserve.Schema.SPREAD_PRIME_CORE
            );            
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L561-L564
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L682
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L686
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L957-L962

```solidity
Use nested if statements instead of &&
file: Tokens/Prime/PrimeLiquidityProvider.sol

217  uint256 balance = token_.balanceOf(address(this));

234 uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

259  uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L217
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L234
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L259

## [G-3]  Use uint256(1)/uint256(2) instead for true and false boolean states
If you don’t use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from true to false can cost up to ~20000 gas rather than uint256(2) to uint256(1) that would cost significantly less.

```solidity

file: Tokens/Prime/Prime.sol

222  isScoreUpdated[nextScoreUpdateRoundId][user] = true;

299    markets[vToken].exists = true;

340  _mint(true, users[i]);

349  for (uint256 i = 0; i < users.length; ) {
                _mint(false, users[i]);

707  tokens[user].exists = true;

750  tokens[user].exists = false;
751  tokens[user].isIrrevocable = false;

765 userToken.isIrrevocable = true;

905    if (amount >= MINIMUM_STAKED_XVS) {
            return true;
909   return false;  

403  _mint(false, msg.sender);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L222
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L299
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L340
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L349
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L403
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L707
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L750-L751
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L765
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L905
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L909

## [G-4] Make 3 event parameters indexed when possible
It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity

file: Tokens/Prime/Prime.sol

51     event Mint(address indexed user, bool isIrrevocable);

    /// @notice Emitted when prime token is burned
    event Burn(address indexed user);

    /// @notice Emitted asset state is update by protocol share reserve
    event UpdatedAssetsState(address indexed comptroller, address indexed asset);

    /// @notice Emitted when a market is added to prime program
60    event MarketAdded(address indexed market, uint256 indexed supplyMultiplier, uint256 indexed borrowMultiplier);

63  event MintLimitsUpdated(
        uint256 indexed oldIrrevocableLimit,
        uint256 indexed oldRevocableLimit,
        uint256 indexed newIrrevocableLimit,
        uint256 newRevocableLimit
    );

    /// @notice Emitted when user score is updated
    event UserScoreUpdated(address indexed user);

    /// @notice Emitted when alpha is updated
    event AlphaUpdated(
        uint128 indexed oldNumerator,
        uint128 indexed oldDenominator,
        uint128 indexed newNumerator,
        uint128 newDenominator
79    );

82    event MultiplierUpdated(
        address indexed market,
        uint256 indexed oldSupplyMultiplier,
        uint256 indexed oldBorrowMultiplier,
        uint256 newSupplyMultiplier,
        uint256 newBorrowMultiplier
    );

    /// @notice Emitted when interest is claimed
    event InterestClaimed(address indexed user, address indexed market, uint256 amount);

    /// @notice Emitted when revocable token is upgraded to irrevocable token
    event TokenUpgraded(address indexed user);

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L51
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L63-L79
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L82-L94

```solidity
()
file: Tokens/Prime/PrimeLiquidityProvider.sol

30     event TokenDistributionInitialized(address indexed token);

    /// @notice Emitted when a new token distribution speed is set
    event TokenDistributionSpeedUpdated(address indexed token, uint256 newSpeed);

    /// @notice Emitted when prime token contract address is changed
    event PrimeTokenUpdated(address oldPrimeToken, address newPrimeToken);

    /// @notice Emitted when distribution state(Index and block) is updated
    event TokensAccrued(address indexed token, uint256 amount);

    /// @notice Emitted when token is transferred to the prime contract
    event TokenTransferredToPrime(address indexed token, uint256 amount);

    /// @notice Emitted on sweep token success
    event SweepToken(address indexed token, address indexed to, uint256 sweepAmount);

    /// @notice Emitted on updation of initial balance for token
    event TokenInitialBalanceUpdated(address indexed token, uint256 balance);

    /// @notice Emitted when funds transfer is paused
    event FundsTransferPaused();

    /// @notice Emitted when funds transfer is resumed
54    event FundsTransferResumed();

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L30-L54

## [G-5] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time the variable is used, which wastes some gas.

```solidity

file: Prime/PrimeStorage.sol

34  uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;

    /// @notice maximum XVS taken in account when calculating user score
    uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

    /// @notice number of days user need to stake to claim prime token
40    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L34-L40

## [G-6] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

```solidity

file: Prime/Prime.sol

692  asset.safeTransfer(user, amount);

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L692

```solidity

file: Prime/PrimeLiquidityProvider.sol

42  event TokenTransferredToPrime(address indexed token, uint256 amount);

204   IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);

224  token_.safeTransfer(to_, amount_);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L42
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L204
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L224

## [G-7] Loop best practice to save gas

```solidity

file: Prime/Prime.sol

739    unchecked {
                i++;
            }

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L739

```solidity

file: Prime/PrimeLiquidityProvider.sol

107 
            unchecked {
                ++i;
            }

122 
            unchecked {
                ++i;
            }

165   unchecked {
                ++i;
            }            
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L107
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L122
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L165

## [G-8] Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: Tokens/Prime/Prime.sol


318  if (_irrevocableLimit < totalIrrevocable || _revocableLimit < totalRevocable) revert InvalidLimit();

716   if (totalIrrevocable > irrevocableLimit || totalRevocable > revocableLimit) revert InvalidLimit();

795    if (!markets[market].exists || !tokens[user].exists) {
            return;
810  if (_alphaDenominator == 0 || _alphaNumerator > _alphaDenominator) {
            revert InvalidAlphaArguments();            
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L318
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L716
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L795
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L810

```solidity

file: Tokens/Prime/libs/Scores.sol

44  if (xvs == 0 || capital == 0) return 0;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/Scores.sol#L44

## [G-9] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: Tokens/Prime/Prime.sol

574  distributionIncome += unreleasedPLPAccruedInterest;

674  amount += interests[vToken][user].accrued;

785  interests[vToken][user].accrued += _interestAccrued(vToken, user);

978  amount += BLOCKS_PER_YEAR * totalIncomePerBlockFromPLP;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L574
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L674
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L785
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L978

```solidity

file: Tokens/Prime/PrimeLiquidityProvider.sol

266  tokenAmountAccrued[token_] += tokenAccrued;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L266

```solidity

file: Tokens/Prime/libs/FixedMath0x.sol

122   r += (z * (0x100000000000000000000000000000000 - y)) / 0x100000000000000000000000000000000;
        z = (z * w) / FIXED_1; // add y^01 / 01 - y^02 / 02
        r += (z * (0x0aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa - y)) / 0x200000000000000000000000000000000;
        z = (z * w) / FIXED_1; // add y^03 / 03 - y^04 / 04
        r += (z * (0x099999999999999999999999999999999 - y)) / 0x300000000000000000000000000000000;
        z = (z * w) / FIXED_1; // add y^05 / 05 - y^06 / 06
        r += (z * (0x092492492492492492492492492492492 - y)) / 0x400000000000000000000000000000000;
        z = (z * w) / FIXED_1; // add y^07 / 07 - y^08 / 08
        r += (z * (0x08e38e38e38e38e38e38e38e38e38e38e - y)) / 0x500000000000000000000000000000000;
        z = (z * w) / FIXED_1; // add y^09 / 09 - y^10 / 10
        r += (z * (0x08ba2e8ba2e8ba2e8ba2e8ba2e8ba2e8b - y)) / 0x600000000000000000000000000000000;
        z = (z * w) / FIXED_1; // add y^11 / 11 - y^12 / 12
        r += (z * (0x089d89d89d89d89d89d89d89d89d89d89 - y)) / 0x700000000000000000000000000000000;
        z = (z * w) / FIXED_1; // add y^13 / 13 - y^14 / 14
136        r +

164    r += z * 0x10e1b3be415a0000; // add y^02 * (20! / 02!)
        z = (z * y) / FIXED_1;
        r += z * 0x05a0913f6b1e0000; // add y^03 * (20! / 03!)
        z = (z * y) / FIXED_1;
        r += z * 0x0168244fdac78000; // add y^04 * (20! / 04!)
        z = (z * y) / FIXED_1;
        r += z * 0x004807432bc18000; // add y^05 * (20! / 05!)
        z = (z * y) / FIXED_1;
        r += z * 0x000c0135dca04000; // add y^06 * (20! / 06!)
        z = (z * y) / FIXED_1;
        r += z * 0x0001b707b1cdc000; // add y^07 * (20! / 07!)
        z = (z * y) / FIXED_1;
        r += z * 0x000036e0f639b800; // add y^08 * (20! / 08!)
        z = (z * y) / FIXED_1;
        r += z * 0x00000618fee9f800; // add y^09 * (20! / 09!)
        z = (z * y) / FIXED_1;
        r += z * 0x0000009c197dcc00; // add y^10 * (20! / 10!)
        z = (z * y) / FIXED_1;
        r += z * 0x0000000e30dce400; // add y^11 * (20! / 11!)
        z = (z * y) / FIXED_1;
        r += z * 0x000000012ebd1300; // add y^12 * (20! / 12!)
        z = (z * y) / FIXED_1;
        r += z * 0x0000000017499f00; // add y^13 * (20! / 13!)
        z = (z * y) / FIXED_1;
        r += z * 0x0000000001a9d480; // add y^14 * (20! / 14!)
        z = (z * y) / FIXED_1;
        r += z * 0x00000000001c6380; // add y^15 * (20! / 15!)
        z = (z * y) / FIXED_1;
        r += z * 0x000000000001c638; // add y^16 * (20! / 16!)
        z = (z * y) / FIXED_1;
        r += z * 0x0000000000001ab8; // add y^17 * (20! / 17!)
        z = (z * y) / FIXED_1;
        r += z * 0x000000000000017c; // add y^18 * (20! / 18!)
        z = (z * y) / FIXED_1;
        r += z * 0x0000000000000014; // add y^19 * (20! / 19!)
        z = (z * y) / FIXED_1;
200        r += z * 0x0000000000000001; // add y^20 * (20! / 20!)

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L122-L136
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L164-L

## [G‑10] Counting down in for statements is more gas efficient
Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

```solidity

file: Tokens/Prime/Prime.sol

178   for (uint256 i = 0; i < _allMarkets.length; )

204  for (uint256 i = 0; i < users.length; )

211  for (uint256 j = 0; j < _allMarkets.length; ) 

246  for (uint256 i = 0; i < allMarkets.length; )

335   for (uint256 i = 0; i < users.length; )

349  for (uint256 i = 0; i < users.length; ) 

609  for (uint256 i = 0; i < _allMarkets.length; ) 

625  for (uint256 i = 0; i < _allMarkets.length; ) 

730  for (uint256 i = 0; i < _allMarkets.length; )
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L178
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L204
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L211
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L335
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L349
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L609
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L625
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L730

```solidity

file: Prime/PrimeLiquidityProvider.sol

103  for (uint256 i; i < numTokens; )

119  for (uint256 i; i < tokens_.length; )

161  for (uint256 i; i < numTokens; ) 
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L103
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L119
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L161

## [G‑11] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity

file: Tokens/Prime/Prime.sol

178   for (uint256 i = 0; i < _allMarkets.length; )

204  for (uint256 i = 0; i < users.length; )

211  for (uint256 j = 0; j < _allMarkets.length; ) 

246  for (uint256 i = 0; i < allMarkets.length; )

335   for (uint256 i = 0; i < users.length; )

349  for (uint256 i = 0; i < users.length; ) 

609  for (uint256 i = 0; i < _allMarkets.length; ) 

625  for (uint256 i = 0; i < _allMarkets.length; ) 

730  for (uint256 i = 0; i < _allMarkets.length; )
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L178
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L204
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L211
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L335
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L349
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L609
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L625
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L730

```solidity

file: Prime/PrimeLiquidityProvider.sol

103  for (uint256 i; i < numTokens; )

119  for (uint256 i; i < tokens_.length; )

161  for (uint256 i; i < numTokens; ) 
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L103
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L119
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L161

## [G-12] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: Tokens/Prime/Prime.sol

228  emit UserScoreUpdated(user);

241  emit AlphaUpdated(alphaNumerator, alphaDenominator, _alphaNumerator, _alphaDenominator);

269   emit MultiplierUpdated

308  emit MarketAdded(vToken, supplyMultiplier, borrowMultiplier);

320    emit MintLimitsUpdated(irrevocableLimit, revocableLimit, _irrevocableLimit, _revocableLimit);

462  emit UpdatedAssetsState(comptroller, asset);

694  emit InterestClaimed(user, vToken, amount);

718   emit Mint(user, isIrrevocable);

755  emit Burn(user);

771      emit TokenUpgraded(user);
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L228
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L241
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L269
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L308
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L320
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L462
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L694
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L718
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L755
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L771


```solidity

file: Prime/PrimeLiquidityProvider.sol#

180  emit PrimeTokenUpdated(prime, prime_);

202  emit TokenTransferredToPrime(token_, accruedAmount);

222   emit SweepToken(address(token_), to_, amount_);

267  emit TokensAccrued(token_, tokenAccrued);

300  emit TokenDistributionInitialized(token_);

324  emit TokenDistributionSpeedUpdated(token_, distributionSpeed_);

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L180
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L202
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L222
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L267
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L300
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L324

## [G-13]  Use selfbalance() instead of address(this).balance


```solidity

file: okens/Prime/Prime.sol

541   _reentrancyStatus = true(protocolShareReserve).getUnreleasedFunds(
            comptroller,
            IProtocolShareReserve.Schema.SPREAD_PRIME_CORE,
            address(this),

682     if (amount > asset.balanceOf(address(this)))  

586  if (amount > asset.balanceOf(address(this))) {
                IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
                unreleasedPLPIncome[underlying] = 0;
            }
957    function _distributionPercentage() internal view returns (uint256) {
        return
            IProtocolShareReserve(protocolShareReserve).getPercentageDistribution(
                address(this),
                IProtocolShareReserve.Schema.SPREAD_PRIME_CORE
            );            
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L561-L564
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L682
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L686
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L957-L962

```solidity
Use nested if statements instead of &&
file: Tokens/Prime/PrimeLiquidityProvider.sol

217  uint256 balance = token_.balanceOf(address(this));

234 uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

259  uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L217
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L234
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L259

## [G-14] Non efficient zero initialization
Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

```solidity

file: Tokens/Prime/Prime.sol

178   for (uint256 i = 0; i < _allMarkets.length; )

204  for (uint256 i = 0; i < users.length; )

211  for (uint256 j = 0; j < _allMarkets.length; ) 

246  for (uint256 i = 0; i < allMarkets.length; )

335   for (uint256 i = 0; i < users.length; )

349  for (uint256 i = 0; i < users.length; ) 

609  for (uint256 i = 0; i < _allMarkets.length; ) 

625  for (uint256 i = 0; i < _allMarkets.length; ) 

730  for (uint256 i = 0; i < _allMarkets.length; )
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L178
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L204
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L211
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L335
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L349
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L609
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L625
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L730

```solidity

file: Prime/PrimeLiquidityProvider.sol

103  for (uint256 i; i < numTokens; )

119  for (uint256 i; i < tokens_.length; )

161  for (uint256 i; i < numTokens; ) 
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L103
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L119
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L161

## [G-15] Don't initialize variables with default value

```solidity

file: Tokens/Prime/Prime.sol

295    markets[vToken].rewardIndex = 0;

298   markets[vToken].sumOfMembersScore = 0;

376  stakedAt[user] = 0;

401   stakedAt[msg.sender] = 0;

460  unreleasedPSRIncome[_getUnderlying(address(market))] = 0;

677    interests[vToken][user].accrued = 0;

688   unreleasedPLPIncome[underlying] = 0;

736  interests[_allMarkets[i]][user].score = 0;
            interests[_allMarkets[i]][user].rewardIndex = 0;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L295
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L298
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L376
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L401
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L460
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L677
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L688
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L736-L737

## [G-16] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements

Saves 5 gas per iteration

```solidity

file: Tokens/Prime/Prime.sol

188    unchecked {
                i++;
            }

216    unchecked {
                    j++;
                }         

216  unchecked {
                i++;
            }      

249        unchecked {
                i++;
            }
        }          
344 
                unchecked {
                    i++;
                }      
354    unchecked {
                    i++;
                } 

613 
            unchecked {
                i++;
            }         

635  unchecked {
                i++;
            }     

711 totalIrrevocable++;

 
 713  totalRevocable++;     

 739   unchecked {
                i++;
            }    
766   totalIrrevocable++;
767   totalRevocable--;    

819 nextScoreUpdateRoundId++;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L188
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L216
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L224
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L249
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L344
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L354
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L613
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L635
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L711-L713
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L739
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L766-L767
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L819

```solidity

file: Prime/PrimeLiquidityProvider.sol

107    unchecked {
                ++i;
            }

122    unchecked {
                ++i;
            }      
165    unchecked {
                ++i;
            }                  
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L107
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L122
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L165

## [G-17]  Not using the named return variable when a function returns, wastes deployment gas

```solidity

file: Tokens/Prime/Prime.so

193   return pendingInterests;

470  return allMarkets;

696   return amount;

856 return MAXIMUM_XVS_CAP;

858 return xvs;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L193
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L470
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L696
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L856
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L858

## [G-18] Using storage instead of memory for structs/arrays saves gas


```solidity

file: Tokens/Prime/Prime.sol

174  function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
      
176        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

200  function updateScores(address[] memory users) external 

469  function getAllMarkets() external view returns (address[] memory) {
        return allMarkets;
683  address[] memory assets = new address[](1);        
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L176
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L469
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L683

## [G-19] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity

file: Tokens/Prime/Prime.sol

554 function accrueInterest(address vToken) public 

597  function getInterestAccrued(address vToken, address user) public returns (uint256) 
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L554
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L597

```solidity

file: Prime/PrimeLiquidityProvider.sol

249 function accrueTokens(address token_) public

276  function getBlockNumber() public view virtual returns (uint256) 
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L276