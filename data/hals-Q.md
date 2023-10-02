# QA Report

| ID            | Title                                                                                       | Severity |
| ------------- | ------------------------------------------------------------------------------------------- | -------- |
| [L-01](#l-01) | Deprecated markets can't be removed from `allMarkets` array                                 | Low      |
| [L-02](#l-02) | `Prime.updateScores` will revert if users are added after updating `nextScoreUpdateRoundId` | Low      |
| [L-03](#l-03) | Prime contract doesn't support vTokens of decimals > 18                                     | Low      |
| [L-04](#l-04) | No check on the `exchangeRate ` before calculationg user score                              | Low      |
| [L-05](#l-05) | Owner of `PrimeLiquidityProvider` can sweep registerd (initialized) tokens                  | Low      |
| [L-06](#l-06) | The project uses vulnerable node dependencies                                               | Low      |
| [L-07](#l-07) | Rewards of an asset blockliste user will be stuck in the `Prime`contract                    | Low      |

# Low

## [L-01] Deprecated markets can't be removed from `allMarkets` array <a id="l-01" ></a>

## Vulnerability Details

- In `Prime` contract: markets can be added to prime program by adding the market vToken with it's configuration (supplyMultiplier & borrowMultiplier).

- Before adding a new market; a check is made on the length of the `allMarkets` array via ` _ensureMaxLoops(allMarkets.length)` function , where the array length is checked against a maximum value that was set upon contract initialization (`_setMaxLoopsLimit(_loopsLimit)`).

- With the current mechanism adopted by the protocol; if a prime supported market is deprecated , then the rewards distribution will stop, and this can be achieved by setting the borrow and supply multipliers to 0 and by setting the distribution speeds to zero in the `PrimeLiquidityProvider` contract.

- But deprecated markets will not be removed from the `allMarkets` array in order to keep this market listed so that users that haven't claimed their rewards before market deprecation will still be able to claim them.

# Impact

This will lead to the `allMarket` array grows in size in the future (as the deprecated markets are not removed from it) until it reaches it's maximum limit; which will disable adding any new market in the future.

## Proof of Concept

[Prime.initilize/Line 164](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L164)

```solidity
 _setMaxLoopsLimit(_loopsLimit);
```

[Prime.addMARKET](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L288-L309)

```solidity
    function addMarket(address vToken, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
        _checkAccessAllowed("addMarket(address,uint256,uint256)");
        if (markets[vToken].exists) revert MarketAlreadyExists();

        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);
        if (!isMarketExist) revert InvalidVToken();

        markets[vToken].rewardIndex = 0;
        markets[vToken].supplyMultiplier = supplyMultiplier;
        markets[vToken].borrowMultiplier = borrowMultiplier;
        markets[vToken].sumOfMembersScore = 0;
        markets[vToken].exists = true;

        vTokenForAsset[_getUnderlying(vToken)] = vToken;

        allMarkets.push(vToken);
        _startScoreUpdateRound();

        _ensureMaxLoops(allMarkets.length);

        emit MarketAdded(vToken, supplyMultiplier, borrowMultiplier);
    }
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Add a mechanism to track deprecated markets to enable users from claiming their rewards of these markets; and remove these deprecated markets from the `allMarkets` array so it won't reach its maximum limit in the future.

## [L-02] `Prime.updateScores` will revert if users are added after updating `nextScoreUpdateRoundId` <a id="l-02" ></a>

## Vulnerability Details

- In `Prime` contract: the `updateScores` function is meant to update scores of a batch of users when a new round is started (when the `nextScoreUpdateRoundId` is increased).

- A new round is started (by increasing the `nextScoreUpdateRoundId` variable by one) whenever the admin updates the alpha or any market multipliers, so that the scores of every user needs to be updated as these parameters can modify in a substantial way the calculated scores.

- A new round is started by calling the `_startScoreUpdateRound`; where the `nextScoreUpdateRoundId` is increased by one and the `pendingScoreUpdates` is set equal to the current number of token holders (totalIrrevocable + totalRevocable).

- Then the `updateScores` function can be called **by anyone** to update the scores of a batch of users in all markets; so when a user score is updated; then this user is flagged as being updated by setting `isScoreUpdated[nextScoreUpdateRoundId][user]=true` so it won't be updated again in the same round via the same function.

- Also the `pendingScoreUpdates` is decreased by one for each updated user.

# Impact

- The problem arises when new users are added to the Prime program by minting them prime token; and then these users scores are updated in the same round via `updateScores` function (as this function can be invoked by anyone), but since these users are not accounted for in the `pendingScoreUpdates` variable that were set before starting the round; then this will lead to disabling the `updateScores` function for the current round as the `pendingScoreUpdates` will be eventually equal to zero which will lead to reverting `updateScores` function after then (this function will be disabled for this round).

- These new users scores will be updated after minting them prime token via `_initializeMarkets`; but they will still be able to get their scores updated again via `updateScores` function.

- demonstartion:

1. `_startScoreUpdateRound` function is called after market updates where the `totalIrrevocable` token holders are 3 users, and `totalRevocable` token holders are 2 users; so the `pendingScoreUpdates` will be set to 5.
2. a new 4 users were able to claim revocable tokens after satisfying the prime conditions; so the `totalRevocable` will be increased to 6, and the total number of users would be 9.
3. the `updateScores` function is called on a 5 users at first; this will set the `pendingScoreUpdates` to zero.
4. then the `updateScores` function is called agin on the remaining 4 users (randomly not specifically the ones that were registered after the round start); so the function will revert as the `pendingScoreUpdates` can't be < 0.

- This will disable this function for the current round, and the remaining users that haven't got their scores updated will do it manually by calling `accrueInterestAndUpdateScore` function and update the score for **each** market.

## Proof of Concept

[Prime.\_startScoreUpdateRound](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L818-L822)

```solidity
   function _startScoreUpdateRound() internal {
        nextScoreUpdateRoundId++;
        totalScoreUpdatesRequired = totalIrrevocable + totalRevocable;
        pendingScoreUpdates = totalScoreUpdatesRequired;
    }
```

[Prime.updateScores/Line 221](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L221)

```solidity
pendingScoreUpdates--;
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

In `_initializeMarkets` function :add a flag to indicate that the user got their scores updated when they were added to the Prime program so that they can't pass the check in the `updateScores` function:

```diff
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
+       isScoreUpdated[nextScoreUpdateRoundId][account]=true;
    }
```

## [L-03] Prime contract doesn't support vTokens of decimals > 18<a id="l-03" ></a>

## Vulnerability Details

- In `Prime` contract: `_calculateScore` function is intended to calculate the user score that will be used to calculate his rewards based on his supply and borrow.

- After the user capital is calculated based on the underlying token price of the market and the user's supply and borrow; this value is scaled based on the vToken decimals:

  ```solidity
  capital = capital * (10 ** (18 - vToken.decimals()));
  ```

- The vToken decimals is set equal to the underlying asset that it represents; so if the underlying asset decimals is > 18; then the capital scaling calculation (18 - vToken.decimals()) will revert.

# Impact

- So if the `vToken.decimals()` is > 18; then the `_calculateScore` function will revert and all the functions implementing it will be disabled.

## Proof of Concept

[Prime.\_calculateScore/Line 660](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L660)

```solidity
capital = capital * (10 ** (18 - vToken.decimals()));
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Update `_calculateScore` function to account for vTokens with decimals greater than 18.

## [L-04] No check on the `exchangeRate` before calculationg user score<a id="l-04" ></a>

## Vulnerability Details

- In `Prime` contract: `_calculateScore` function is intended to calculate the user score that will be used to calculate his rewards based on his supply and borrow.

- The supply is calculated based on the exchangeRate from the underlying asset to the vToken:

  ```solidity
  uint256 exchangeRate = vToken.exchangeRateStored();
  ```

  ```solidity
  uint256 supply = (exchangeRate * balanceOfAccount) / EXP_SCALE;
  ```

- And this calculated supply is used to calculate the capital of the user (along with the borrow value); which is the **the sum of qualified supply and borrow balance of the user**.

- But as there's no check on the value of the `exchangeRate` that's extracted from the vToken; then a zero value could be returned (at some cases); so the calculation of the capital would be incorrect (lesser than actual).

# Impact

- This will result in decreasing the user score which will affect his rewards calculation.

## Proof of Concept

[Prime.\_calculateScore/Line 652](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L652)

```solidity
uint256 exchangeRate = vToken.exchangeRateStored();
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Update `_calculateScore` function to revert if the returnd `vToken.exchangeRateStored()` is zero.

## [L-05] Owner of `PrimeLiquidityProvider` can sweep initialized tokens <a id="l-05" ></a>

## Vulnerability Details

In `PrimeLiquidityProvider` contract: the owner can sweep accidental ERC-20 transferred to the contract by calling the `sweepToken` function; but as can be noticed; there's no check if the sweeped token is initialized ;i.e. intended to be used to provide funds for the prime rewards program.

# Impact

So if an initialized token that is meant to fund the prime rewards is **fully** sweeped from the `PrimeLiquidityProvider`; then
the `PrimeLiquidityProvider.accrueTokens` will be disabled as it will revert when checking the `balanceDiff` when the token started to accrue rewards ( tokenAmountAccrued[token_] > 0):
[PrimeLiquidityProvider.accrueTokens / Line 259-261](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L259-L261)

    ```solidity
                uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

                uint256 balanceDiff = balance - tokenAmountAccrued[token_];
    ```

leading to disabling all the functionalities of the Prime feature for this sweeped token market where `PrimeLiquidityProvider.accrueTokens` is called.

## Proof of Concept

[PrimeLiquidityProvider.sweepToken](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225)

```solidity
    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
        uint256 balance = token_.balanceOf(address(this));
        if (amount_ > balance) {
            revert InsufficientBalance(amount_, balance);
        }

        emit SweepToken(address(token_), to_, amount_);

        token_.safeTransfer(to_, amount_);
    }
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Update `PrimeLiquidityProvider.sweepToken` function to prevent sweeping initialized (registered) tokens.

## [L-06] The project uses vulnerable node dependencies<a id="l-06" ></a>

## Vulnerability Details

The Protocol uses multiple third-party dependencies; some of these were affected by public-known vulnerabilities that may pose a risk
to the global application security level.

# Impact

This may pose a risk to the global application security level.

## Proof of Concept

```shell
# npm audit report

@openzeppelin/contracts  <=4.8.2
Severity: high
OpenZeppelin Contracts initializer reentrancy may lead to double initialization - https://github.com/adviso
Improper Initialization in OpenZeppelin - https://github.com/advisories/GHSA-88g8-f5mf-f5rj
OpenZeppelin Contracts TransparentUpgradeableProxy clashing selector calls may not be delegated - https://g2q-35m2-x2rh
OpenZeppelin Contracts ERC165Checker unbounded gas consumption - https://github.com/advisories/GHSA-7grf-83
fix available via `npm audit fix`
node_modules/@openzeppelin/contracts-v0.7

flat  <5.0.1
Severity: critical
flat vulnerable to Prototype Pollution - https://github.com/advisories/GHSA-2j2x-2gpw-g8fm
fix available via `npm audit fix`
node_modules/eth-gas-reporter/node_modules/flat
  yargs-unparser  <=1.6.3
  Depends on vulnerable versions of flat
  node_modules/eth-gas-reporter/node_modules/yargs-unparser
    mocha  5.1.0 - 9.2.1
    Depends on vulnerable versions of minimatch
    Depends on vulnerable versions of yargs-unparser
    node_modules/eth-gas-reporter/node_modules/mocha
      eth-gas-reporter  0.0.5 - 0.2.25
      Depends on vulnerable versions of mocha
      Depends on vulnerable versions of request
      Depends on vulnerable versions of request-promise-native
      node_modules/eth-gas-reporter

minimatch  <3.0.5
Severity: high
minimatch ReDoS vulnerability - https://github.com/advisories/GHSA-f8q6-p94x-37v3
fix available via `npm audit fix`
node_modules/eth-gas-reporter/node_modules/minimatch

request  *
Severity: moderate
Server-Side Request Forgery in Request - https://github.com/advisories/GHSA-p8p7-x288-28g6
Depends on vulnerable versions of tough-cookie
fix available via `npm audit fix`
node_modules/request
  request-promise-core  *
  Depends on vulnerable versions of request
  node_modules/request-promise-core
    request-promise-native  >=1.0.0
    Depends on vulnerable versions of request
    Depends on vulnerable versions of request-promise-core
    Depends on vulnerable versions of tough-cookie
    node_modules/request-promise-native

semver  7.0.0 - 7.5.1
Severity: moderate
semver vulnerable to Regular Expression Denial of Service - https://github.com/advisories/GHSA-c2qf-rxjj-qq
fix available via `npm audit fix --force`
Will install @semantic-release/npm@11.0.0, which is a breaking change
node_modules/npm/node_modules/semver
  npm  7.0.0-beta.0 - 9.7.1
  Depends on vulnerable versions of semver
  node_modules/npm
    @semantic-release/npm  7.1.0 - 10.0.0-beta.4
    Depends on vulnerable versions of npm
    node_modules/@semantic-release/npm
      semantic-release  18.0.0-beta.1 - 20.1.3
      Depends on vulnerable versions of @semantic-release/npm
      node_modules/semantic-release

tough-cookie  <4.1.3
Severity: moderate
tough-cookie Prototype Pollution vulnerability - https://github.com/advisories/GHSA-72xf-g2v4-qvf3
fix available via `npm audit fix`
node_modules/tough-cookie

14 vulnerabilities (8 moderate, 3 high, 3 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Update all affected packages to its latest version.

## [L-07] Rewards of an asset blocklisted user will be stuck in the `Prime` contract <a id="l-07" ></a>

## Vulnerability Details

- In `Prime._claimInterest` function: user can claim his rewards tokens from any listed market directly to his address; but some tokens have a **blocklist** where certain accounts are prohibited from receiving or transferring any tokens.

- So if the user is set in the blocklist of the asset (market) token (if it has a blacklist); then his reward tokens will be stuck in the prime contract .

# Impact

Blocklisted user in the underlying market asset can't get his accrued rewards.

## Proof of Concept

[Prime.claimInterest](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L433-L435)

```solidity
   function claimInterest(address vToken) external whenNotPaused returns (uint256) {
        return _claimInterest(vToken, msg.sender);
    }
```

[Prime.claimInterest](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L443-L445)

```solidity
  function claimInterest(address vToken, address user) external whenNotPaused returns (uint256) {
        return _claimInterest(vToken, user);
    }
```

[Prime.\_claimInterest](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L672-L697)

```solidity
function _claimInterest(address vToken, address user) internal returns (uint256) {
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

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Update `claimInterest` & `_claimInterest` functions to have third parameter (receiver) where the user wants to send his rewards tokens.

```diff
- function claimInterest(address vToken, address user) external whenNotPaused returns (uint256) {
+ function claimInterest(address vToken, address user,address receiver) external whenNotPaused returns (uint256) {
-       return _claimInterest(vToken, user);
+       return _claimInterest(vToken, user,receiver);
    }
```

```diff
function _claimInterest(address vToken, address user,address receiver) internal returns (uint256) {
      //some code
-       asset.safeTransfer(user, amount);
+       asset.safeTransfer(receiver, amount);
      //some code
    }
```
