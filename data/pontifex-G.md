### GAS-1 Using redundant caching
There are 4 instances.
Using the `distributionSpeed` variable for caching is redundant.
```solidity
233        uint256 distributionSpeed = tokenDistributionSpeeds[token_];
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L233
Using the `blockNumber` and `initializedBlock` variables for caching is redundant.
```solidity
288        uint256 blockNumber = getBlockNumber();
289        uint256 initializedBlock = lastAccruedBlock[token_];
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L288-L298
Using the `lastBlockAccrued` variable for caching is redundant.
```solidity
333        uint256 lastBlockAccrued = lastAccruedBlock[token_];
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L333
PrimeLiquidityProvider.sol




### GAS-2 Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time the variable is used, which wastes some gas.
There are 3 instances:
```solidity
34    uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;


37    uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;


40    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L34




### GAS-3 Some functions can be turned into a pure function
These functions do not read from storage. They can therefore be changed from view to pure to save on gas.
There are __ instances:
```solidity
854    function _xvsBalanceForScore(uint256 xvs) internal view returns (uint256) {
855        if (xvs > MAXIMUM_XVS_CAP) {
856            return MAXIMUM_XVS_CAP;
857        } else {
858            return xvs;
859        }
860    }
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L854-L860
```solidity
904    function isEligible(uint256 amount) internal view returns (bool) {
905        if (amount >= MINIMUM_STAKED_XVS) {
906            return true;
907        }
908
909        return false;
910    }
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L904-L910




### G-4 Double checks
The `_` check is the same as the `_` modifier.
The `_ensureTokenInitialized` check is called twice during the `setTokensDistributionSpeed` function execution: at the line L#162 and again at the line L#252 (_setTokenDistributionSpeed -> accrueTokens).
Consider removing the check at the line L#162.
```solidity
162            _ensureTokenInitialized(tokens_[i]);
163            _setTokenDistributionSpeed(tokens_[i], distributionSpeeds_[i]);            
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L162




### G-5 Multiple rechecks
There are 2 instances of multiple rechecking:
The `if (!tokens[user].exists)` check at the lines L#780 and L#795 duplicates the same check from the `updateScores` function which call `_executeBoost` and `_updateScore`. Consider removing this check from `_executeBoost` and `_updateScore` function and adding it at the `accrueInterestAndUpdateScore` and `_accrueInterestAndUpdateScore` functions with corresponding logic.
```solidity
200    function updateScores(address[] memory users) external {


204        for (uint256 i = 0; i < users.length; ) {


207            if (!tokens[user].exists) revert UserHasNoPrimeToken();


211            for (uint256 j = 0; j < _allMarkets.length; ) {


213                _executeBoost(user, market);
214                _updateScore(user, market);




779    function _executeBoost(address user, address vToken) internal {
780        if (!markets[vToken].exists || !tokens[user].exists) {


794    function _updateScore(address user, address market) internal {
795        if (!markets[market].exists || !tokens[user].exists) {
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L200-L214




### GAS-6 Pre increment/decrement costs less gas than post increment/decrement or `<x> += 1` and `<x> -= 1`
Pre increment/decrement costs less than post increment decrement/decrement or `<x> += 1` and `<x> -= 1`. It can save more than 5 gas per execution.
There are 19 instances:
```solidity
189                i++;


217                    j++;


221            pendingScoreUpdates--;


225                i++;


250                i++;


345                    i++;


355                    i++;


614                i++;


636                i++;


711            totalIrrevocable++;


713            totalRevocable++;


740                i++;


745            totalIrrevocable--;


747            totalRevocable--;


766        totalIrrevocable++;
767        totalRevocable--;


819        nextScoreUpdateRoundId++;


828        if (totalScoreUpdatesRequired > 0) totalScoreUpdatesRequired--;


831            pendingScoreUpdates--;
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L189




### G-7 Redundant multiplications and divisions
There are redundant divisions by the `EXP_SCALE` constant at the lines L#881-882, 885-886 which have no effect on the return value:
```solidity
881        uint256 borrowCapUSD = (xvsPrice * ((xvs * markets[market].borrowMultiplier) / EXP_SCALE)) / EXP_SCALE;
882        uint256 supplyCapUSD = (xvsPrice * ((xvs * markets[market].supplyMultiplier) / EXP_SCALE)) / EXP_SCALE;


885        uint256 supplyUSD = (tokenPrice * supply) / EXP_SCALE;
886        uint256 borrowUSD = (tokenPrice * borrow) / EXP_SCALE;


888        if (supplyUSD >= supplyCapUSD) {
889            supply = supplyUSD > 0 ? (supply * supplyCapUSD) / supplyUSD : 0;
890        }


892        if (borrowUSD >= borrowCapUSD) {
893            borrow = borrowUSD > 0 ? (borrow * borrowCapUSD) / borrowUSD : 0;
894        }
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L881-L893
The proposed implementation:
```solidity
881        uint256 borrowCapUSD = xvsPrice * ((xvs * markets[market].borrowMultiplier) / EXP_SCALE);
882        uint256 supplyCapUSD = xvsPrice * ((xvs * markets[market].supplyMultiplier) / EXP_SCALE);


885        uint256 supplyUSD = (tokenPrice * supply);
886        uint256 borrowUSD = (tokenPrice * borrow);


888        if (supplyUSD >= supplyCapUSD) {
889            supply = supplyUSD > 0 ? (supply * supplyCapUSD) / supplyUSD : 0;
890        }


892        if (borrowUSD >= borrowCapUSD) {
893            borrow = borrowUSD > 0 ? (borrow * borrowCapUSD) / borrowUSD : 0;
894        }
```


### G-8 Consider function optimization
Consider the `xvsUpdated` function optimization to improve both readability and gas economy.
The current implementation:
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L365-L382
```solidity
    function xvsUpdated(address user) external {
        uint256 totalStaked = _xvsBalanceOfUser(user);
        bool isAccountEligible = isEligible(totalStaked);


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
    }
```
The proposed implementation:
```solidity
    function xvsUpdated(address user) external {
        uint256 totalStaked = _xvsBalanceOfUser(user);
        bool isAccountEligible = isEligible(totalStaked);


        if (tokens[user].exists) {
            if (isAccountEligible || tokens[user].isIrrevocable) {
                _accrueInterestAndUpdateScore(user);
            } else {
               _burn(user);
            }
        } else if (stakedAt[user] == 0 && isAccountEligible) {
            stakedAt[user] = block.timestamp;
        } else if (!isAccountEligible && stakedAt[user] > 0) {
            stakedAt[user] = 0;
        }
    }
```






### GAS-9 Move `if`/validation statements as up as possible in the function body
`if (distributionSpeed > 0)` can be above `balance` variable declaration
```solidity
258            uint256 distributionSpeed = tokenDistributionSpeeds[token_];
259            uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));


261            uint256 balanceDiff = balance - tokenAmountAccrued[token_];
262            if (distributionSpeed > 0 && balanceDiff > 0) {
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L258-L262




### GAS-10 Redundant pointers declaration
There are 5 instances
```solidity
175        address[] storage _allMarkets = allMarkets;


210            address[] storage _allMarkets = allMarkets;


608        address[] storage _allMarkets = allMarkets;


624        address[] storage _allMarkets = allMarkets;


728        address[] storage _allMarkets = allMarkets;
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L175
