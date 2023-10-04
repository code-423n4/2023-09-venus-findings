### Report 1:
No Proper error handling from denial of service when total Revocable is zero which will revert the code from the "totalRevocable--;" subtraction operation.
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L767
```solidity
 function _upgrade(address user) internal {
        Token storage userToken = tokens[user];
+++ if ( totalRevocable == 0) revert InvalidRevocation();
        userToken.isIrrevocable = true;
        totalIrrevocable++;
        totalRevocable--;

        if (totalIrrevocable > irrevocableLimit) revert InvalidLimit();

        emit TokenUpgraded(user);
    }
```
As noted above proper error handling should be added to the code depending on sponsors preference other instances can be found at [L568](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L568)
### Report 2:
No Upper limit when setting _alphaDenominator, which could cause DOS without proper error handling.
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L809-L813
```solidity
    function _checkAlphaArguments(uint128 _alphaNumerator, uint128 _alphaDenominator) internal 
+++     if ( _alphaDenominator > _alphaDenominatorLimit ) { // a limit of preference should be added to code
+++            revert InvalidAlphaArguments();
+++        }
        if (_alphaDenominator == 0 || _alphaNumerator > _alphaDenominator) {
            revert InvalidAlphaArguments();
        }
    }
```



