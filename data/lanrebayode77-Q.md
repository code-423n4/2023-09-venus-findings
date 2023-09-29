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

### 2. Possible Incorrect BlocksPerYear
https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L109
``` solidity
 BLOCKS_PER_YEAR = _blocksPerYear;
```
Setting the BlocksPerYear in the constructor without check is error prone.
BlocksPerYear = 365 * 24 * 60 * 60 / 12 = 2_628_000

If the BLOCKS_PER_YEAR is set incorrectly, _incomeDistributionYearly will be calculated wrongly. https://github.com/code-423n4/2023-09-venus/blob/edc2212c77c8a419bd49a05ec1e2556405095922/contracts/Tokens/Prime/Prime.sol#L970-L979
``` solidity
 function _incomeDistributionYearly(address vToken) internal view returns (uint256 amount) {
        uint256 totalIncomePerBlockFromMarket = _incomePerBlock(vToken);
        uint256 incomePerBlockForDistributionFromMarket = (totalIncomePerBlockFromMarket * _distributionPercentage()) /
            IProtocolShareReserve(protocolShareReserve).MAX_PERCENT();
        amount = BLOCKS_PER_YEAR * incomePerBlockForDistributionFromMarket;

        uint256 totalIncomePerBlockFromPLP = IPrimeLiquidityProvider(primeLiquidityProvider)
            .getEffectiveDistributionSpeed(_getUnderlying(vToken));
        amount += BLOCKS_PER_YEAR * totalIncomePerBlockFromPLP;
    }
```

For total assurance, there should be
``` solidity
require(_blocksPerYear == 2_628_000, "INCORRECT BLOCKS");
```