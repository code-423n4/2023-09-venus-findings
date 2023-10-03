# Low

# Summary
|      |  issue  | insteance |
|------|---------|-----------|
|[L-01]| Unprotected Setter in `Prime.setLimit` Function|2|
|[L-02]| Unprotected Initializer in `Prime._initializeMarkets` Function|2|
|[L-03]| Dangerous Strict Equality Check in `Prime` Contract's `Claim` Function|6|
|[L-04]| Initialize Local Variable in `Prime.accrueInterest` Function|3|
|[L-05]| Infinite Loop Risk in `Prime` Contract's `updateScores` Function|1|
|[L-06]| Reentrancy Risk in `Prime` Contract's `updateAlpha` Function|6|
|[L-07]| Reentrancy Risk in `Prime` Contract's `issue` Function|4|
|[L-08]| Cautionary Usage of block.timestamp in Prime Contract's claim Function|2|
|[L-09]| Emitting Events in PrimeLiquidityProvider's setTokensDistributionSpeed Function|1|

## [L-01] Unprotected Setter in `Prime.setLimit` Function
The function setLimit(uint256,uint256) in Prime.sol (lines 316-324) lacks access control, posing a security risk. This non-protected setter, revocableLimit, allows modification without proper authorization.

```solidity
    function setLimit(uint256 _irrevocableLimit, uint256 _revocableLimit) external {
        _checkAccessAllowed("setLimit(uint256,uint256)");
        if (_irrevocableLimit < totalIrrevocable || _revocableLimit < totalRevocable) revert InvalidLimit();

        emit MintLimitsUpdated(irrevocableLimit, revocableLimit, _irrevocableLimit, _revocableLimit);

        revocableLimit = _revocableLimit;
        irrevocableLimit = _irrevocableLimit;
    }
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L316-L324

`Recommendation`:
To enhance security, it is recommended to add access control to the `setLimit` function. This ensures that only authorized users can modify the limits, aligning with best practices for smart contract development.

There is 1 other instance of this issue:

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L237-L255


## [L-02] Unprotected Initializer in `Prime._initializeMarkets` Function
The function Prime._initializeMarkets(address) at lines 623-639 is identified as an unprotected initializer. Initializers play a crucial role in the smart contract lifecycle, and their exposure without proper protection poses a security risk.

```solidity
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
`Recommendation`:

To enhance the security of the `Prime` contract, it is strongly advised to protect initializers with appropriate modifiers or require statements. This ensures that the function is only executed under controlled and authorized conditions, mitigating the potential risks associated with unprotected initializers.

There is 1 other instance of this issue:
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L286-L301

## [L-03] Dangerous Strict Equality Check in `Prime` Contract's `Claim` Function
The Prime.claim() function, specifically at line 398 (stakedAt[msg.sender] == 0), utilizes a dangerous strict equality check. This practice can introduce vulnerabilities, making it susceptible to manipulation by potential attackers.

```solidity
398  if (stakedAt[msg.sender] == 0) revert IneligibleToClaim();
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L398

`Recommendation`:

It is recommended to avoid using strict equality checks, especially when determining an account's eligibility based on Ether or tokens. Instead, consider using range-based checks or other secure comparison methods to mitigate potential exploitation.

There is 5 other instance of this issue:
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L479

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L1002

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L1007

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L1012

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L335

## [L-04] Initialize Local Variable in `Prime.accrueInterest` Function
The `Prime.accrueInterest(address).delta` at line 583 involves a local variable, `delta`, that is never explicitly initialized in the function. Uninitialized variables can lead to unexpected behavior and may introduce bugs or vulnerabilities.

```solidity
583  uint256 delta;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L583

`Recommendation`:
To enhance code clarity and prevent potential issues, it is recommended to explicitly initialize all local variables. If a variable is intended to be initialized to zero, set it to zero at the beginning of the function.

There is 2 other instance of this issue:

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L65

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L159

## [L-05] Infinite Loop Risk in `Prime` Contract's `updateScores` Function
The function `Prime.updateScores(address[])` contains a vulnerable for loop structure, where a `continue` statement precedes an unchecked index increment. This pattern can potentially lead to an infinite loop, posing a significant risk to the contract's functionality and user experience.

Code Snippet:
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230

`Recommendation`:
To eliminate the risk of an infinite loop, it is crucial to increment the loop index before the continue statement, ensuring that the loop progresses correctly.

## [L-06] Reentrancy Risk in `Prime` Contract's `updateAlpha` Function
The function `Prime.updateAlpha(uint128,uint128)` exhibits a potential reentrancy vulnerability involving external calls to `accrueInterest(allMarkets[i])` and `_primeLiquidityProvider.accrueTokens(underlying)`. Following these external calls, state variables are modified, creating a risk of reentrancy exploitation.

Code Snippet:
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L237-L255

Recommendation:

To mitigate the reentrancy risk, it is recommended to apply the [check-effects-interactions](http://solidity.readthedocs.io/en/v0.4.21/security-considerations.html#re-entrancy) pattern. Ensure that state modifications occur before any external calls to prevent unexpected interactions.

Implementation Suggestion:

```solidity
function updateAlpha(uint128 _alphaNumerator, uint128 _alphaDenominator) external {
    // Existing implementation details

    for (uint256 i = 0; i < allMarkets.length; ) {
        // Move state modification before external call
        unchecked {
            i++;
        }
        accrueInterest(allMarkets[i]);
    }

    _startScoreUpdateRound();
}

```
By adhering to the`check-effects-interactions `pattern and modifying state variables before external calls, you reduce the risk of reentrancy issues and enhance the overall security of the `updateAlpha` function.

There is 5 other instance of this issue:

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L263-L280

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L794-L802

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L779-L787

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L725-L756

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L554-L589

## [L-07] Reentrancy Risk in `Prime` Contract's `issue` Function
The `Prime.issue(bool, address[])` function exhibits a potential reentrancy vulnerability due to external calls to `_initializeMarkets(users[i])`, `_primeLiquidityProvider.accrueTokens(underlying)`, `oracle.updateAssetPrice(xvsToken)`, and `oracle.updatePrice(market)`. Events, including `Mint(user, isIrrevocable)`, `_mint(true, users[i])`, `TokenUpgraded(user)`, and `_upgrade(users[i])`, are emitted after these external calls, introducing the risk of manipulation in the order or value of events.

Code Snippet:
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359

`Recommendation`:

To mitigate the reentrancy risk and ensure the proper order of events, it is recommended to apply the [check-effects-interactions](https://docs.soliditylang.org/en/latest/security-considerations.html#re-entrancy) pattern. Organize the code to perform state modifications before external calls, especially those that emit events.

There is 3 other instance of this issue:

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L725-L756

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L672-L697

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L263-L280

## [L-08] Cautionary Usage of block.timestamp in Prime Contract's claim Function
The `Prime.claim()`function utilizes `block.timestamp` in comparisons (`stakedAt[msg.sender] == 0` and `block.timestamp - stakedAt[msg.sender] < STAKING_PERIOD`). This usage is considered risky, as `block.timestamp` can be manipulated by miners, potentially leading to undesired outcomes.

Code Snippet:
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L397-L405

`Recommendation`:
To enhance security and avoid reliance on `block.timestamp`, it is recommended to use alternative methods for time-related operations. Consider using block numbers or an external time oracle to mitigate the risk of miner manipulation.

There is 1 other instance of this issue:
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L478-L487

## [L-09] Emitting Events in PrimeLiquidityProvider's setTokensDistributionSpeed Function
The `PrimeLiquidityProvider.setTokensDistributionSpeed` function is identified as a setter function that lacks event emission. According to best practices, setter functions should emit events to provide transparency and allow monitoring of state changes.

Code Snippet:
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L153-L169

`Recommendation`:
To align with best practices and enhance transparency, it is recommended to emit events within setter functions. Emitting events communicates state changes and provides valuable information for monitoring and auditing.

