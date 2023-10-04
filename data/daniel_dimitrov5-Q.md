1. [Prime.sol#L904](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L904), isEligible can be restricted to pure and can be simplified
**Propose**
```solidity
    function isEligible(uint256 amount) internal pure returns (bool) {
        return amount >= MINIMUM_STAKED_XVS;
    }
```
2. Significant version difference between different parts of the Venus protocol.
e.g. https://github.com/code-423n4/2023-09-venus/blob/main/contracts/XVSVault/XVSVault.sol#L1
3. [Prime.sol#L306](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L306) addMarket. call _ensureMaxLoops after _startScoreUpdateRound. Consider calling _ensureMaxLoops before _startScoreUpdateRound, because the length check is in _ensureMaxLoops.
4. [Prime.sol#200](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200) Prime.sol updateScores may run out of gas when called many users. Use _ensureMaxLoops to make sure the loop is bounded
5. Prime.sol - There is no mechanism for removing markets. Consider adding one just in case.
6. storage variables can be restricted to memory since the state is not modified:
   1. [Prime.sol#175](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L175)
   2. [Prime.sol#210](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L210)
   3. [Prime.sol#336](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L336)
   4. [Prime.sol#607](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L607)
   5. [Prime.sol#624](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L624)
   6. [Prime.sol#728](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L728)
   7. [Prime.sol#763](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L763)

7. _checkAlphaArguments and _xvsBalanceForScore in Prime.sol can be restricted to pure.
