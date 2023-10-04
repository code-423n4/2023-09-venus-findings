## Low Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | User can lose prime token issued by governance if he deposits/withdraws any amount from vault | 1 |
| [L-2](#L-2) | vToken.decimals() should validate if token decimal is not greater than 18 | 1 |

## Non-Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Important functions are missing event emission | 2 |

## [L-1] User can lose prime token issued by governance if he deposits/withdraws any amount from vault

Prime Tokens (revokable or irrevocable) can be directly issued to the users by the governance by calling the `issue()` function without passing the eligibility criteria of maintaining a balance of 1000 tokens for a period of 90 days. Meaning even if a user doesn't have a minimum of 1000 tokens, they can be granted a prime token.
However, if such a user decides to deposit or withdraw any amount into their account using the functions `deposit()` or `requestWithdrawal` available in the XVSVault, they will lose (burn) their Revocable Prime Token because the said vault functions call the `xvsUpdated()` in the Prime contract.

```solidity
File:  Prime.sol

     function xvsUpdated(address user) external {
        uint256 totalStaked = _xvsBalanceOfUser(user);
        bool isAccountEligible = isEligible(totalStaked);    /// @note - false balance is less than 1000 tokens

        if (tokens[user].exists && !isAccountEligible) {
            if (tokens[user].isIrrevocable) {
                _accrueInterestAndUpdateScore(user);
            } else {
                _burn(user);                                /// @note - users token will get burned
         }
```

## [L-2] vToken.decimals() should validate if token decimal is not greater than 18 

Checking the decimal of the token to be greater than 18 early on in the function can prevent unnecessary calculations & errors.

```solidity
File: Prime.sol

        function _calculateScore(address market, address user) internal returns (uint256) {
        uint256 xvsBalanceForScore = _xvsBalanceForScore(_xvsBalanceOfUser(user));

        IVToken vToken = IVToken(market);
        uint256 borrow = vToken.borrowBalanceStored(user);
        uint256 exchangeRate = vToken.exchangeRateStored();
        uint256 balanceOfAccount = vToken.balanceOf(user);
        uint256 supply = (exchangeRate * balanceOfAccount) / EXP_SCALE;

        address xvsToken = IXVSVault(xvsVault).xvsAddress();
        oracle.updateAssetPrice(xvsToken);
        oracle.updatePrice(market);

        (uint256 capital, , ) = _capitalForScore(xvsBalanceForScore, borrow, supply, market);
        capital = capital * (10 ** (18 - vToken.decimals()));                                     /// @audit - should be checked in the beginning

        return Scores.calculateScore(xvsBalanceForScore, capital, alphaNumerator, alphaDenominator);
    }
```
## [NC-1] Important functions are missing event emission
`accrueInterestAndUpdateScore()` accrues interest and updates score for an user for a specific market but no event is emitted providing any info on the new interest & updated score.

```solidity
File: Prime.sol

    function accrueInterestAndUpdateScore(address user, address market) external {
        _executeBoost(user, market);
        _updateScore(user, market);
    }
```

Whenever a prime token is minted `_initializeMarkets()` is called after `_mint()`. The mint function does emit an event alerting the user that a prime token has been minted but the user is unaware of the score they have obtained.
Thus, `_initializeMarkets()` should also emit an event displaying the values contained inside the struct Interest, that is accrued, score & rewardIndex.

```solidity
File: Prime.sol

    function _initializeMarkets(address account) internal {
        address[] storage _allMarkets = allMarkets;
        for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            accrueInterest(market);

            interests[market][account].rewardIndex = markets[market].rewardIndex;

            uint256 score = _calculateScore(market, account);
            interests[market][account].score = score;
            markets[market].sumOfMembersScore = markets[market].sumOfMembersScore + score;

            unchecked {
                i++;
            }
        }
    }
```