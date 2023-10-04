### Report 1:
No Proper error handling from denial of service when total Revocable is zero
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
As noted above proper error handling should be added to the code depending on sponsors preference

