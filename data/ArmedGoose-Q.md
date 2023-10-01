*L-01 There is no possibility to remove a market*
Markets are added by a trusted entity. Once added, markets stay in the Prime token forever, and in most operations all markets are iterated through (there is no possibility to skip)
In some rare cases, e.g. if a market related to certain token might be delisted for example if underlying token is subject to a hack, and its price becomes unstable, it might be better to remove that market from being accounted. However in current state it is impossible to do so.
*Recommendation*: Add removeMarket function.

*L-02 Claiming for other users may be against their will*
The protocol in `Prime.sol` line 443 has an ability to claim interest using ``function claimInterest(address vToken, address user)` without restriction. That means, that anyone can make others claim their interest. While there is no direct loss of funds, it could be against someone's will to e.g. claim at specified point of time. 

```solidity 
    function claimInterest(address vToken, address user) external whenNotPaused returns (uint256) {
        return _claimInterest(vToken, user); //@audit-issue why for other user?
    }
```
*Recommendation*: Add a logic to allow someone to claim on behalf or allo only direct claim for users themselves.

*I-01 Function can be defined as pure*
In `Prime.sol`, line 809, function `_checkAlphaArguments` can be defined as pure as it does only operations on user input and a revert. 

```solidity
    function _checkAlphaArguments(uint128 _alphaNumerator, uint128 _alphaDenominator) internal {
        if (_alphaDenominator == 0 || _alphaNumerator > _alphaDenominator) {
            revert InvalidAlphaArguments();
        }
    }
```

*Recommendation*: Define function as `function _checkAlphaArguments(uint128 _alphaNumerator, uint128 _alphaDenominator) internal pure`.