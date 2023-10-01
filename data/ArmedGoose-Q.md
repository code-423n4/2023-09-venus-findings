*L-01 There is no possibility to remove a market*
Markets are added by a trusted entity. Once added, markets stay in the Prime token forever, and in most operations all markets are iterated through (there is no possibility to skip)
In some rare cases, e.g. if a market related to certain token might be delisted for example if underlying token is subject to a hack, and its price becomes unstable, it might be better to remove that market from being accounted. However in current state it is impossible to do so.
*Recommendation*: Add removeMarket function.

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