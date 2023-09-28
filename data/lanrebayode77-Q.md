## QA REPORT

### 1. Use View for function non-changing state function
https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L597-L601
The function does not change any state, just returns a uin256
``` solidity
    function getInterestAccrued(address vToken, address user) public returns (uint256) {
        accrueInterest(vToken);

        return _interestAccrued(vToken, user);
    }
```
Use view.
``` solidity
    function getInterestAccrued(address vToken, address user) public view returns (uint256) {
        accrueInterest(vToken);

        return _interestAccrued(vToken, user);
    }
```