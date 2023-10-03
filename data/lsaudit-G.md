# [G-01] `calculateScore` in `libs/Scores.sol` can be simplified

[File: contracts/Tokens/Prime/libs/Scores.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/libs/Scores.sol#L52-L68)
```
        bool lessxvsThanCapital = xvs < capital;

        // (xvs / capital) or (capital / xvs), always in range (0, 1)
        int256 ratio = lessxvsThanCapital ? FixedMath.toFixed(xvs, capital) : FixedMath.toFixed(capital, xvs);

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
```

* There's no need to declare `bool lessxvsThanCapital`, `int256 ratio`, `int256 exponentiation`.
* There's no need to check `xvs < capital` condition twice (first at line 55, then at line 62).

Above line of code can be simplified to:

```
    if (xvs < capital) {
       
        return FixedMath.uintMul(capital, FixedMath.exp((FixedMath.ln(FixedMath.toFixed(xvs, capital)) * alphaNumerator.toInt256()) / alphaDenominator.toInt256()));
    }
     
    return FixedMath.uintDiv(capital, FixedMath.exp((FixedMath.ln(FixedMath.toFixed(capital, xvs)) * alphaNumerator.toInt256()) / alphaDenominator.toInt256()));
```

That way you'll be saving gas on declaring less variables (`ratio` and `exponentiation` values are directly in the `return` statement). Also, you'll be saving gas on not checking `xvs < capital` condition twice. This condition is being checked only once.


# [G-02] Use `delete` keyword, instead of assigning mapping's key to 0.

A short test of resetting mapping's key to 0 has been performed.
We've estimated how much gas it will take to set the value by using `= 0` and by using `delete`:

```
function setToZero() public {
	uint a = gasleft();
	lastAccruedBlock[msg.sender] = 0;
	console.log(a - gasleft());
}

function useDelete() public {
	uint a = gasleft();
	delete lastAccruedBlock[msg.sender] ;
	console.log(a - gasleft() );

}
```

It turned out, that `setToZero()` used 5113 gas, while `useDelete()`: 5108.
This implies, that `delete` is more gas efficient.

This means that [PrimeLiquidityProvider.sol#200](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L200) can be changed, from:
```
tokenAmountAccrued[token_] = 0;
```
to

```
delete tokenAmountAccrued[token_];
```

The same issue occurs in [Prime.sol#295](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L295-L298), where:

```
        markets[vToken].rewardIndex = 0;
        markets[vToken].supplyMultiplier = supplyMultiplier;
        markets[vToken].borrowMultiplier = borrowMultiplier;
        markets[vToken].sumOfMembersScore = 0;
```

can be changed to:

```
        delete markets[vToken].rewardIndex;
        markets[vToken].supplyMultiplier = supplyMultiplier;
        markets[vToken].borrowMultiplier = borrowMultiplier;
        delete markets[vToken].sumOfMembersScore;
```

The same issue occurs in [Prime.sol#376](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L376), where:
```
 stakedAt[user] = 0;
```
can be changed to:
```
 delete stakedAt[user];
```

The same issue occurs in [Prime.sol#401](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L401), where:
```
stakedAt[msg.sender] = 0;
```
can be changed to:
```
delete stakedAt[msg.sender];
```

The same issue occurs in [Prime.sol#460](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L460), where:
```
 unreleasedPSRIncome[_getUnderlying(address(market))] = 0;
```
can be changed to:

```
 delete unreleasedPSRIncome[_getUnderlying(address(market))];
```

The same issue occurs in [Prime.sol#688](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L688), where:
```
unreleasedPLPIncome[underlying] = 0;
```
can be changed to:
```
delete unreleasedPLPIncome[underlying];
```

The same issue occurs in [Prime.sol#736-737](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L736-L751), where:
```
            interests[_allMarkets[i]][user].score = 0;
            interests[_allMarkets[i]][user].rewardIndex = 0;
            (...)
            tokens[user].exists = false;
            tokens[user].isIrrevocable = false;
```
can be changed to:

```
           delete interests[_allMarkets[i]][user].score;
           delete interests[_allMarkets[i]][user].rewardIndex;
           (...)
            delete tokens[user].exists;
            delete tokens[user].isIrrevocable;
```

# [G-03] Precalculate constants

[File: contracts/Tokens/Prime/PrimeStorage.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L34-L43)
```
uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;

/// @notice maximum XVS taken in account when calculating user score
uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

/// @notice number of days user need to stake to claim prime token
uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;
```

Values of constants can be calculated before compilation time.

# [G-04] Unnecessary variable declaration in `getEffectiveDistributionSpeed` in `PrimeLiquidityProvider.sol`
Variables `distributionSpeed `, `balance`, `accrued` are used only once, which means, that below code:

[File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L232-L242)
```
    function getEffectiveDistributionSpeed(address token_) external view returns (uint256) {
        uint256 distributionSpeed = tokenDistributionSpeeds[token_];
        uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
        uint256 accrued = tokenAmountAccrued[token_];

        if (balance - accrued > 0) {
            return distributionSpeed;
        }

        return 0;
    }
```

can be rewritten to:

```
    function getEffectiveDistributionSpeed(address token_) external view returns (uint256) {
        if (IERC20Upgradeable(token_).balanceOf(address(this)) - tokenAmountAccrued > 0) {
            return tokenDistributionSpeeds[token_];
        }

        return 0;
    }
```

# [G-05] Unnecessary variable declaration in `_initializeToken` in `PrimeLiquidityProvider.sol`
Variable `blockNumber`, `initializedBlock` are used only once, which means, that below code:

[File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L286-L301)
```
    function _initializeToken(address token_) internal {
        _ensureZeroAddress(token_);
        uint256 blockNumber = getBlockNumber();
        uint256 initializedBlock = lastAccruedBlock[token_];

        if (initializedBlock > 0) {
            revert TokenAlreadyInitialized(token_);
        }

        /*
         * Update token state block number
         */
        lastAccruedBlock[token_] = blockNumber;

        emit TokenDistributionInitialized(token_);
    }
```

can be rewritten to:

```
    function _initializeToken(address token_) internal {
        _ensureZeroAddress(token_);

        if (lastAccruedBlock[token_] > 0) {
            revert TokenAlreadyInitialized(token_);
        }

        /*
         * Update token state block number
         */
        lastAccruedBlock[token_] = getBlockNumber();

        emit TokenDistributionInitialized(token_);
    }
```

# [G-06] Unnecessary variable declaration in `_ensureTokenInitialized` in `PrimeLiquidityProvider.sol`
Variable `lastBLockAccrued` is used only once, which means, that below code:

[File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L332-338)
```
    function _ensureTokenInitialized(address token_) internal view {
        uint256 lastBlockAccrued = lastAccruedBlock[token_];

        if (lastBlockAccrued == 0) {
            revert TokenNotInitialized(token_);
        }
    }
```

can be rewritten to:

```
    function _ensureTokenInitialized(address token_) internal view {
        if (lastAccruedBlock[token_] == 0) {
            revert TokenNotInitialized(token_);
        }
    }
```

# [G-07] Use `unchecked` keyword for calculations which won't underflow in `PrimeLiquidityProvider.sol`

[File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L254-L255)
```
        uint256 blockNumber = getBlockNumber();
        uint256 deltaBlocks = blockNumber - lastAccruedBlock[token_];
```

Function `getBlockNumber()` returns the current `block.number`. Since `lastAccruedBlock[token_]` constains previous `block.number` (it's assigned at line 207 of function `accrueTokens`), we can be sure, that this calculation won't underflow. The current `block.number` is always bigger than the previous one - thus this expression can be `unchecked`.


# [G-08] Unnecessary variable declaration in `addMarket` in `Prime.sol`
Variable `isMarketExist` is used only once, which means that below code:

[File: contracts/Tokens/Prime/Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L292-L293)
```
        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);
        if (!isMarketExist) revert InvalidVToken();
```

can be rewritten to:

```
if (!InterfaceComptroller(comptroller).markets(vToken)) revert InvalidVToken();

```

# [G-09] Unnecessary variable declaration in `xvsUpdated` in `Prime.sol`
Variable `totalStaked` is used only once, which means that below code:

[File: contracts/Tokens/Prime/Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L366-367)
```
uint256 totalStaked = _xvsBalanceOfUser(user);
bool isAccountEligible = isEligible(totalStaked);
```
can be rewritten to:

```
bool isAccountEligible = isEligible(_xvsBalanceOfUser(user));
```

# [G-10] Calculations which won't underflow can be inserted into `unchecked` block in `claimTimeRemaining` in `Prime.sol`:

[File: contracts/Tokens/Prime/Prime.so](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L482-L483)
```
if (totalTimeStaked < STAKING_PERIOD) {
            return STAKING_PERIOD - totalTimeStaked;
```
Since `totalTimeStaked < STAKING_PERIO`, then `STAKING_PERIOD - totalTimeStaked` will never underflow and can be `unchecked`.


# [G-11] Unnecessary variable declaration in `calculateAPR`, `_calculateScore` in `Prime.sol`
Variables `exchangeRate` and `balanceOfAccount` are used only once, which means that below code:

[File: contracts/Tokens/Prime/Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L498-L501)
```
        uint256 exchangeRate = vToken.exchangeRateStored();
        uint256 balanceOfAccount = vToken.balanceOf(user);
        uint256 supply = (exchangeRate * balanceOfAccount) / EXP_SCALE;
```

can be rewritten to:

```
        uint256 supply = (vToken.exchangeRateStored() * vToken.balanceOf(user)) / EXP_SCALE;
```

The same issue occurs for `_calculateScore`:

[File: contracts/Tokens/Prime/Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L652-L654)
```
        uint256 exchangeRate = vToken.exchangeRateStored();
        uint256 balanceOfAccount = vToken.balanceOf(user);
        uint256 supply = (exchangeRate * balanceOfAccount) / EXP_SCALE;
```

can be rewritten to:

```
uint256 supply = (vToken.exchangeRateStored() *  vToken.balanceOf(user)) / EXP_SCALE;
```

# [G-12] Unnecessary variable declaration in `_capitalForScore` in `Prime.sol`
Variable `xvsToken` is used only once, which means that below code:

[File: contracts/Tokens/Prime/Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L878-L880)
```
address xvsToken = IXVSVault(xvsVault).xvsAddress();

uint256 xvsPrice = oracle.getPrice(xvsToken);
```

can be rewritten to:

```
uint256 xvsPrice = oracle.getPrice(IXVSVault(xvsVault).xvsAddress());
```

# [G-13] Unnecessary variable declaration in `_interestAccrued` in `Prime.sol`
Variables `index` and `score` are used only once, which means, that below code:
[File: contracts/Tokens/Prime/Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L918-L923)
```
    function _interestAccrued(address vToken, address user) internal view returns (uint256) {
        uint256 index = markets[vToken].rewardIndex - interests[vToken][user].rewardIndex;
        uint256 score = interests[vToken][user].score;

        return (index * score) / EXP_SCALE;
    }
```

can be rewritten to:
```
    function _interestAccrued(address vToken, address user) internal view returns (uint256) {
        return ((markets[vToken].rewardIndex - interests[vToken][user].rewardIndex) * interests[vToken][user].score) / EXP_SCALE;
    }
```

# [G-13] Unnecessary variable declaration in `_interestAccrued` in `Prime.sol`
Variables `index` and `score` are used only once, which means, that below code:
[File: contracts/Tokens/Prime/Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L971-L974)
```
  uint256 totalIncomePerBlockFromMarket = _incomePerBlock(vToken);
        uint256 incomePerBlockForDistributionFromMarket = (totalIncomePerBlockFromMarket * _distributionPercentage()) /
            IProtocolShareReserve(protocolShareReserve).MAX_PERCENT();
        amount = BLOCKS_PER_YEAR * incomePerBlockForDistributionFromMarket;
```

can be rewritten to:

```
        amount = BLOCKS_PER_YEAR * (_incomePerBlock(vToken) * _distributionPercentage()) /
            IProtocolShareReserve(protocolShareReserve).MAX_PERCENT();
```

# [G-14] `if`-condition can be moved up in `_calculateUserAPR` in `Prime.sol`

[File: contracts/Tokens/Prime/Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L1004-1007)
```
        uint256 userYearlyIncome = (userScore * _incomeDistributionYearly(vToken)) / totalScore;
        uint256 totalCappedValue = totalCappedSupply + totalCappedBorrow;

        if (totalCappedValue == 0) return (0, 0);
```

You can calculate `userYearlyIncome` value after `totalCappedValue`. That way, if `totalCappedValue == 0`, you will return with `(0, 0)` sooner, without wasting gas on `userYearlyIncome` calculations:

```
        uint256 totalCappedValue = totalCappedSupply + totalCappedBorrow;
        if (totalCappedValue == 0) return (0, 0);
        uint256 userYearlyIncome = (userScore * _incomeDistributionYearly(vToken)) / totalScore;
```

# [G-15] Make 3 event parameters indexed when possible

* [File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol)
```
event TokenDistributionSpeedUpdated(address indexed token, uint256 newSpeed);
event PrimeTokenUpdated(address oldPrimeToken, address newPrimeToken);
event TokensAccrued(address indexed token, uint256 amount);
event TokenTransferredToPrime(address indexed token, uint256 amount);
event SweepToken(address indexed token, address indexed to, uint256 sweepAmount);
event TokenInitialBalanceUpdated(address indexed token, uint256 balance);
```

* [File: contracts/Tokens/Prime/Prime.sol](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol)
```
event Mint(address indexed user, bool isIrrevocable);
event InterestClaimed(address indexed user, address indexed market, uint256 amount);
```

# [G-16] Do-while loops are cheaper than for loops

Using do-while loops are better for saving gas than for-loops.

Multiple of for-loops can be rewritten to do-while loops.

* File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol
```
103:    for (uint256 i; i < numTokens; ) {
119:    for (uint256 i; i < tokens_.length; ) {
161:    for (uint256 i; i < numTokens; ) {
```

* File: contracts/Tokens/Prime/Prime.sol
```
178:    for (uint256 i = 0; i < _allMarkets.length; ) {
204:    for (uint256 i = 0; i < users.length; ) {
211:    for (uint256 j = 0; j < _allMarkets.length; ) {
246:    for (uint256 i = 0; i < allMarkets.length; ) {
335:    for (uint256 i = 0; i < users.length; ) {
349:    for (uint256 i = 0; i < users.length; ) {
609:    for (uint256 i = 0; i < _allMarkets.length; ) {
625:    for (uint256 i = 0; i < _allMarkets.length; ) {
730:    for (uint256 i = 0; i < _allMarkets.length; ) {
```

# [G-17] Use ++var instead of var++ to increment

The reason behind this is in way `++var` and `var++` are evaluated by the compiler.

Using `var++` - 2 values are stored on the stack (`var++` returns `var` (the old value) before incrementing to a new value).
Using `++var` - 1 value is stored on the stack  (`++var` evaluates `++` operator on `var` and then returns `var`).
This means that `++var` is more gas effective than `var++`. The same occurs for `var--`/`--var`.

* File: contracts/Tokens/Prime/Prime.sol
```
189:                  i++;
217:                      j++;
225:                  i++;
250:                  i++;
345:                      i++;
355:                      i++;
614:                  i++;
636:                  i++;
711:              totalIrrevocable++;
713:              totalRevocable++;
740:                  i++;
766:          totalIrrevocable++;
819:          nextScoreUpdateRoundId++;

221:              pendingScoreUpdates--;
745:              totalIrrevocable--;
747:              totalRevocable--;
767:          totalRevocable--;
828:          if (totalScoreUpdatesRequired > 0) totalScoreUpdatesRequired--;
831:              pendingScoreUpdates--;
```

