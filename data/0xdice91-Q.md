`Low Risk & Non Critical Findings 5`
# QA Report



**Low-Risk Findings List**



| Number | Issue Detail | Severity |
|-----:|----|-----|
| [`L-01`](#L-01-`totalScoreUpdatesRequired`-is-not-reduced-after-each-iteration-in-`updateScores()`.) | `totalScoreUpdatesRequired` is not reduced after each iteration in `updateScores()` | Low |
| [`L-02`](#L-02-`claimTimeRemaining()`-is-not-implemented-correctly) |`claimTimeRemaining()` is not implemented correctly. | Low |
| [`L-03`](#L-03-Missing-function-to-change-the-minimum-XVS-token-required-for-`Prime-Token`.) | Missing function to change the minimum XVS token required for `Prime Token`. | Low |
| [`L-04`](#L-04-`getEffectiveDistributionSpeed()`-should-check-if-the-token-has-been-initialised.) | `getEffectiveDistributionSpeed()` should check if the token has been initialised. | Low |
| [`L-05`](#L-05-`_claimInterest()`-should-check-if-the-user-has-a-`Prime-Token`-first.) | `_claimInterest()` should check if the user has a `Prime Token` first. | Low |



> Total ~ 5 Issues

# Low Risk Findings 

## L-01 `totalScoreUpdatesRequired` is not reduced after each iteration in `updateScores()`.
### Summary 
[`totalScoreUpdatesRequired`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L93-L94) is a variable used to track the total number of accounts whose score needs to be updated. When updating the scores for each user this variable needs to be reduced as well to ensure proper accounting, but in the function [`updateScores()`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L200-L230) this is not done leading to improper tracking of the variable.

### Vulnerability Details 
The variable in question;
```solidity
    /// @notice total number of accounts whose score needs to be updated
    uint256 public totalScoreUpdatesRequired;
```
In the function `updateScores()`, `totalScoreUpdatesRequired` is not reduced after a score has been updated.
```solidity
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


            pendingScoreUpdates--;
            isScoreUpdated[nextScoreUpdateRoundId][user] = true;


            unchecked {
                i++;
            }


            emit UserScoreUpdated(user);
        }
    }
```


### Recommended Mitigation Step.
The variable `totalScoreUpdatesRequired` should be reduced after each interaction.
```solidity
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


            pendingScoreUpdates--;
            totalScoreUpdatesRequired--;
            isScoreUpdated[nextScoreUpdateRoundId][user] = true;


            unchecked {
                i++;
            }


            emit UserScoreUpdated(user);
        }
    }
```








## L-02 `claimTimeRemaining()` is not implemented correctly
### Summary 
The function [`claimTimeRemaining()`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L478-L487) is implemented correctly as it returns the wrong output for if (stakedAt[user] == 0).

### Vulnerability Details 
When the (stakedAt[user] == 0) this signifies that the user is not eligible for `Prime Token` that is their XVS balance is below 1000 tokens.
but the function treats (stakedAt[user] == 0) as if the staker is eligible by returning the `STAKING_PERIOD`.
```solidity
    function claimTimeRemaining(address user) external view returns (uint256) {
        if (stakedAt[user] == 0) return STAKING_PERIOD;


        uint256 totalTimeStaked = block.timestamp - stakedAt[user];
        if (totalTimeStaked < STAKING_PERIOD) {
            return STAKING_PERIOD - totalTimeStaked;
        } else {
            return 0;
        }
    }
```


### Recommended Mitigation Step.
The function should rearranged to return `not eligible` if the (stakedAt[user] == 0).







## L-03 Missing function to change the minimum XVS token required for `Prime Token`.
### Summary 
Cryptocurrencies are very volatile and as the protocol evolves great changes in the price of the XVS Token can occur which will affect Venus Prime if it is not built to adapt to these unexpected changes.

### Vulnerability Details 
Venus prime lacks a function to change the `MINIMUM_STAKED_XVS`. As of now XVS is worth roughly 4.5 dollars but with time its value may rise to 150 dollars or fall greatly to be worth a couple of cents, this leads to the 1000 XVS being too high for the average guy or too easy for everyone to get respectively. This will disrupt Venus Prime as it will no longer be able to achieve its aim of increasing user engagement with the protocol.

### Recommended Mitigation Step.
An access-controlled function to change the value of the `MINIMUM_STAKED_XVS` should be added to prime.sol.






## L-04 `getEffectiveDistributionSpeed()` should check if the token has been initialised.
### Summary 
The function does not check if the token has been initialised.


### Vulnerability Details 
The function [`getEffectiveDistributionSpeed()`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L232-L242) used to get rewards per block for a token does not check if the token has been initialised therefore, if an unknown token is inputted as a parameter will return unexpected values.
```solidity
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


### Recommended Mitigation Step.
The check should be added to the function.
```solidity
function getEffectiveDistributionSpeed(address token_) external view returns (uint256) {

        _ensureTokenInitialized(token_);
        uint256 distributionSpeed = tokenDistributionSpeeds[token_];
        uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
        uint256 accrued = tokenAmountAccrued[token_];


        if (balance - accrued > 0) {
            return distributionSpeed;
        }


        return 0;
    }
```





## L-05 `_claimInterest()` should check if the user has a `Prime Token` first.
### Summary 
The internal function `_claimInterest()` should check if the user has a `Prime Token` before executing another logic.


### Vulnerability Details 
In the external `claimInterest()` and the internal `_claimInterest()` functions the user is not checked to ensure that the address has a `Prime Token` holder, this leaves the function open to be called by anyone and furthermore leaves a backdoor for which although not seen now can be exploited by malicious actors to benefit themselves or harm the protocol in the future.


### Recommended Mitigation Step.
Add a check to ensure that the user is a `Prime Token` holder before logic for claiming rewards is executed.
```solidity
    function _claimInterest(address vToken, address user) internal returns (uint256) {
        if (!tokens[user].exists) revert UserHasNoPrimeToken();
        uint256 amount = getInterestAccrued(vToken, user);
        amount += interests[vToken][user].accrued;


        interests[vToken][user].rewardIndex = markets[vToken].rewardIndex;
        interests[vToken][user].accrued = 0;


        address underlying = _getUnderlying(vToken);
        IERC20Upgradeable asset = IERC20Upgradeable(underlying);


        if (amount > asset.balanceOf(address(this))) {
            address[] memory assets = new address[](1);
            assets[0] = address(asset);
            IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
            if (amount > asset.balanceOf(address(this))) {
                IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
                unreleasedPLPIncome[underlying] = 0;
            }
        }


        asset.safeTransfer(user, amount);


        emit InterestClaimed(user, vToken, amount);


        return amount;
    }
```



