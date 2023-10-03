## QA REPORT

### 1. Incorrect natspec comment
Before passing user xvs balance to obtain supply and borrow, it was checked that the XVS balance does not exceed the capped value with an internal function ```_xvsBalanceForScore(xvsStaked)```
``` solidity
 uint256 xvsBalanceForScore = _xvsBalanceForScore(xvsStaked);
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L536-L537
But inside the ```_capitalForScore```, the natspec comment states that xvs passed is the 'actual' balance of user.
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L864
``` solidity
     * @param xvs the actual XVS balance of user
```
It should be corrected to a more clearer comment as the current detail is conflicts what was actually done. A more appropriate comment should be: 
``` solidity
     * @param xvs the capped XVS balance of user
```

### 2. Redundant line of code.
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L156
``` solidity 
nextScoreUpdateRoundId = 0;
```
In solidity unit variables are set to zero by default, the line does not make any changes. 

### 3. Possible Incorrect BlocksPerYear
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

Also, I think this should be a medium severity based on the fact that the protocol documentation states BloksPerYear wrongly in their example to be 10_512_000 blocks/year. This is wrong as that number of blocks will be 4years, so 4X reward will be distributed according to the documentation.: https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/README.md?plain=1#L254
