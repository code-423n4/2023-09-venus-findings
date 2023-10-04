
## Summary
	


|	|	Issue								| Instances	|
| -----|	-----								| -----	|
|[G-01]| Functions guaranteed to revert when called by normal users can be marked payable	|	03	|
|[G-02]|Immutable has more gas efficiency than constant					|	07	|
|[G-03]|Revert inside a loop							|	01	|
|[G-04]|It's cheaper to declare the variable outside the loop				|	06	|






# [G 01] Functions guaranteed to revert when called by normal users can be marked payable
Function with access control marked as payable will be cheaper for legitimate callers: the compiler removes checks for msg.value opcode thus making it more efficient in terms of gas consumption.
Marking the function as payable will lower the gas cost for legitimate callers.
```
File: 2023-09-venus/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

- 118:  function initializeTokens(address[] calldata tokens_) external onlyOwner {
+ L118:     function initializeTokens(address[] calldata tokens_) external onlyOwner payable {

- 177:      function setPrimeToken(address prime_) external onlyOwner {
+ 177:         function setPrimeToken(address prime_) external onlyOwner payable {

- 216:      function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
+ 216:         function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner payable{

```

Link to the Code:
1.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118
2.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177
3.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216


# [G-02] Immutable has more gas efficiency than constant

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.
```
File: 2023-09-venus/contracts/Tokens/Prime/ PrimeStorage.sol

31:    uint256 internal constant EXP_SCALE = 1e18;
34:    uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;
37:    uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;
40:    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;
43:    uint256 internal constant MAXIMUM_BPS = 10000;
```

```
File: 2023-09-venus/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

12:    uint256 public constant MAX_DISTRIBUTION_SPEED = 1e18;
15:    uint256 internal constant EXP_SCALE = 1e18;
```
Variable values when set through constructor using immutable, also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost.

Link to the Code:
1.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L31
2.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L34
3.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L37
4.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L40
5.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L43
6.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L12
7.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L15


# [G-03] Revert inside a loop
Condition inside for loop will carry on, unless it finds one condition that is not met, it will revert the whole transaction. 
It is recommended that instead of reverting if condition is not true, to just break the loop. This way we don't need to run again the transaction.
```
File: 2023-09-venus/contracts/Tokens/Prime/ Prime.sol

- 216:         if (!tokens[user].exists) revert UserHasNoPrimeToken();
+ 216:         if (!tokens[user].exists) break;

```
Link to the Code:
1.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L207


# [G-04] It's cheaper to declare the variable outside the loop
Declaring a variable inside a loop result in variable being re-declared during each loop iteration which consume higher gas.
The variable gets reallocated when declared outside loop making it more gas efficient.
```
File: 2023-09-venus/contracts/Tokens/Prime/ Prime.sol

+	address user;
+	address[] storage _allMarkets;
204: for (uint256 i = 0; i < users.length; ) {
- 205: address user = users[i];
+ 205:	 user = users[i];

if (!tokens[user].exists) revert UserHasNoPrimeToken();
if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;
- 210: address[] storage _allMarkets = allMarkets;
+ 210: _allMarkets = allMarkets;

+ address market;
211: for (uint256 j = 0; j < _allMarkets.length; ) {
- 212:    address market = _allMarkets[j];
+ 212:    address market = _allMarkets[j];


+ Token storage userToken;
335:    for (uint256 i = 0; i < users.length;) {
- 336:		Token storage userToken = tokens[users[i]];
+ 336:		userToken = tokens[users[i]];

+ address market;
+ uint256 score;
625:    for (uint256 i = 0; i < _allMarkets.length; ) {
- 626:            address market = _allMarkets[i];
+ 626:            market = _allMarkets[i];
            accrueInterest(market);
interests[market][account].rewardIndex = markets[market].rewardIndex;
- 336:           uint256 score = _calculateScore(market, account);
+ 336:			score = _calculateScore(market, account);

```

Link to the Code:
1.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L205
2.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L210
3.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L212
4.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L336
5.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L626
6.	https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L631
