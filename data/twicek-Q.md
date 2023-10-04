| Issue ID | Title                                                                                     | Impact                                                                                     | Recommended Mitigation Steps                                                                                           | Code Reference                                                                                                         |
|----------|-------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| L-01     | No remove market function                                                                 | DoS                                                                                        | Consider adding a remove market function.                                                                              | [Prime.sol#L607-L617](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L607-L617)             |
| L-02     | `updateMultipliers` does not have bounds checks for `supplyMultiplier` and `borrowMultiplier` | Limits chance of admin error                                                              | Consider adding bounds limit for `supplyMultiplier` and `borrowMultiplier`.                                            | [Prime.sol#L263-L280](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L263-L280)             |
| L-03     | Consider documenting the processus that prevents the non-issue described in the code README under "Potential Locked Tokens" | Potential for tokens to become stuck in contract if not clearly documented                  | Clarify in documentation and comments the process allowing to prevent this issue.                                       | [Prime.sol#L583-L586](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L583-L586)             |
| N-01     | Adds comments to `_claimInterest` regarding the `PSR.releaseFunds` callback to `updateAssetsState` | Improves code readability                                                                  | Add comments regarding the existence of this callback in `_claimInterest`.                                              | [Prime.sol#L682-L690](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L682-L690)<br>[Prime.sol#L452-L463](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L452-L463) |

# [L-01] No remove market function

## Vulnerability Details
Markets can be added through `addMarket` but cannot be removed. Over time the `allMarkets` array length might become too high to perform certain calls like `_accrueInterestAndUpdateScore`:

[Prime.sol#L607-L617](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L607-L617)
```solidity
    function _accrueInterestAndUpdateScore(address user) internal {
        address[] storage _allMarkets = allMarkets;
        for (uint256 i = 0; i < _allMarkets.length; ) {
            _executeBoost(user, _allMarkets[i]);
            _updateScore(user, _allMarkets[i]);

            unchecked {
                i++;
            }
        }
    }
```

PS: This issue is present in the bot report but with a poor description so I'm adding it here anyway.

## Impact
DoS

## Recommended Mitigation Steps
Consider adding a remove market function.

# [L-02] `updateMultipliers` does not have bounds checks for `supplyMultiplier` and `borrowMultiplier`

## Vulnerability Details
See title.

[Prime.sol#L263-L280](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L263-L280)
```solidity
    function updateMultipliers(address market, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
        _checkAccessAllowed("updateMultipliers(address,uint256,uint256)");
        if (!markets[market].exists) revert MarketNotSupported();


        accrueInterest(market);


        emit MultiplierUpdated(
            market,
            markets[market].supplyMultiplier,
            markets[market].borrowMultiplier,
            supplyMultiplier,
            borrowMultiplier
        );
        markets[market].supplyMultiplier = supplyMultiplier;
        markets[market].borrowMultiplier = borrowMultiplier;


        _startScoreUpdateRound();
    }
```

## Impact
Having bounds for these parameters will limit the chance of an admin error.

## Recommended Mitigation Steps
Consider adding bounds for `supplyMultiplier` and `borrowMultiplier`.

# [L-03] Consider documenting the process that prevents the non-issue described in the code README under "Potential Locked Tokens"

## Vulnerability Details
>Potential Locked Tokens
>According to the logic of function accrueInterest(), it is possible that some rewards will not be collected by >any user. If rewards are currently being issued, but no user has a positive score, then no user can collect >these rewards. Even so, all currently unreleased funds issued can be sent from both PrimeLiquidityProvider and >ProtocolShareReserve to the Prime contract by anyone. Since no user is privy to these funds, they will become >stuck in the contract.

>Prime tokens will be issued at the same time (same transaction) the Prime contracts are enabled, so the described scenario will not happen.

It is not clear what enabling Prime contracts means precisely. If it doesn't mean that admins will wait to provide rewards until there is at least one user with a positive score, it will lead to the issue described.

[Prime.sol#L583-L586](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L583-L586)
```solidity
        uint256 delta;
        if (markets[vToken].sumOfMembersScore > 0) {
            delta = ((distributionIncome * EXP_SCALE) / markets[vToken].sumOfMembersScore);
        }
```

## Impact
If enabling markets solely means to deploy the Prime contract and add markets without having any user with a positive score it could lead to the issue described.

## Recommended Mitigation Steps
Consider clarifying in your documentation and in comments the processus allowing to prevent the issue as it's an important vulnerability surface.

# [N-01] Adds comments to `_claimInterest` regarding the `PSR.releaseFunds` callback to `updateAssetsState`

## Vulnerability Details
In `_claimInterest`, funds are released when there is not enough funds in the contract to cover the claim request. For the PLP funds release it's clear that `unreleasedPLPIncome` is set to 0 after releasing the funds:

[Prime.sol#L682-L690](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L682-L690)
```solidity
        if (amount > asset.balanceOf(address(this))) {
            address[] memory assets = new address[](1);
            assets[0] = address(asset);
            IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
            if (amount > asset.balanceOf(address(this))) {
                IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
                unreleasedPLPIncome[underlying] = 0;
            }
        }
```

But for PSR funds release this update occurs in `updateAssetsState`:
[Prime.sol#L452-L463](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L452-L463)
```solidity
    function updateAssetsState(address _comptroller, address asset) external {
        if (msg.sender != protocolShareReserve) revert InvalidCaller();
        if (comptroller != _comptroller) revert InvalidComptroller();


        address vToken = vTokenForAsset[asset];
        if (vToken == address(0)) revert MarketNotSupported();


        IVToken market = IVToken(vToken);
        unreleasedPSRIncome[_getUnderlying(address(market))] = 0;


        emit UpdatedAssetsState(comptroller, asset);
    }
```

## Recommended Mitigation Steps
Consider adding comments regarding the existence of this callback in `_claimInterest` to improve the code readability.