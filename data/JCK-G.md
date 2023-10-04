
# Gas Optimizations

| Number | Issue | Instances | Total gas saved |
|--------|-------|-----------|-----------------|
|[G-01]| State variables which are not modified within functions should be set as constant or immutable for values set at deployment  | 29 |    |
|[G-02]| Cache external calls outside of loop to avoid re-calling function on each iteration  | 1 |    |
|[G-03]| Use assembly to perform efficient back-to-back calls  | 2 |    |
|[G-04]| Use calldata instead of memory for function arguments that do not get mutated  | 1 | 360 |
|[G-05]| Functions guaranteed to revert when called by normal users can be marked payable  | 2 |    |
|[G-06]| Use hardcoded address instead address(this)  | 7 |    |
|[G-07]| Use uint256(1)/uint256(2) instead for true and false boolean states  | 11 |  188100  |
|[G-08]| Expensive operation inside a for loop  | 2 |    |
|[G-09]| Use assembly to validate msg.sender  | 2 |    |
|[G-10]| Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function  | 8 | 224  |
|[G-11]| Use assembly for loops  | 3 |    |
|[G-12]| Amounts should be checked for 0 before calling a transfer  | 2 |    |
|[G-13]| Should use arguments instead of state variable  | 1 |    |
|[G-14]| Can make the variable outside the loop to save gas  | 9 |    |
|[G-15]| Avoid contract existence checks by using low level calls    | 7 |  700  |
|[G-16]| Using delete statement can save gas  | 13 |    |
|[G-17]| Not using the named return variable when a function returns, wastes deployment gas  | 4 |    |
|[G-18]| Create IVToken,IPrimeLiquidityProvider and IERC20Upgradeable  variable  immutable variable to avoid redundant external calls  | 3 |    |
|[G-19]| Inverting the condition of an if-else-statement wastes gas  | 6 |    |
|[G-20]| Make 3 event parameters indexed when possible  | 7 | 256 |
|[G-21]| Don't initialize variables with default value  | 19 |  114  |
|[G-22]| Duplicated require()/if() checks should be refactored to a modifier or function     | 6 | 168  |
|[G-23]| for mapping use private instead of public if they mapping is used inside they contract   | 3 |  227568  |
|[G-24]| Do not calculate constants   | 3 |    |
|[G-25]| Loop best practice to save gas   | 11 |    |
|[G-26]| Using storage instead of memory for structs/arrays saves gas   | 2 |    |
|[G-27]| Use do while loops instead of for loops   | 6 |    |
|[G-28]| State variables can be packed into fewer storage slots   | 1 |  2100 |
|[G-29]| Move storage pointer to top of function to avoid offset calculation   | 1 |  89  |


## [G-01] State variables which are not modified within functions should be set as constant or immutable for values set at deployment

Cache such variables and perform operations on them, if operations include modifications to the state variable(s) then remember to equate the state variable to it's cached counterpart at the end

```solidity
file:   contracts/Tokens/Prime/PrimeStorage.sol

46       mapping(address => Token) public tokens;

    /// @notice  Tracks total irrevocable tokens minted
49    uint256 public totalIrrevocable;

    /// @notice  Tracks total revocable tokens minted
52    uint256 public totalRevocable;

    /// @notice  Indicates maximum revocable tokens that can be minted
55    uint256 public revocableLimit;

    /// @notice  Indicates maximum irrevocable tokens that can be minted
58    uint256 public irrevocableLimit;

    /// @notice Tracks when prime token eligible users started staking for claiming prime token
61    mapping(address => uint256) public stakedAt;

    /// @notice vToken to market configuration
64    mapping(address => Market) public markets;

    /// @notice vToken to user to user index
67    mapping(address => mapping(address => Interest)) public interests;

    /// @notice A list of boosted markets
70    address[] internal allMarkets;

    /// @notice numberator of alpha. Ex: if alpha is 0.5 then this will be 1
73    uint128 public alphaNumerator;

    /// @notice denominator of alpha. Ex: if alpha is 0.5 then this will be 2
76    uint128 public alphaDenominator;

    /// @notice address of XVS vault
79    address internal xvsVault;

    /// @notice address of XVS vault reward token
82    address internal xvsVaultRewardToken;

    /// @notice address of XVS vault pool id
85    uint256 internal xvsVaultPoolId;

    /// @notice mapping to check if a account's score was updated in the round
88    mapping(uint256 => mapping(address => bool)) public isScoreUpdated;

    /// @notice unique id for next round
91    uint256 public nextScoreUpdateRoundId;

    /// @notice total number of accounts whose score needs to be updated
94    uint256 public totalScoreUpdatesRequired;

    /// @notice total number of accounts whose score is yet to be updated
97    uint256 public pendingScoreUpdates;

    /// @notice mapping used to find if an asset is part of prime markets
100    mapping(address => address) public vTokenForAsset;

    /// @notice address of protocol share reserve contract
103    address public protocolShareReserve;

    /// @notice address of core pool comptroller contract
106    address public comptroller;

    /// @notice unreleased income from PSR that's already distributed to prime holders
    /// @dev mapping of asset adress => amount
110    mapping(address => uint256) public unreleasedPSRIncome;

    /// @notice unreleased income from PLP that's already distributed to prime holders
    /// @dev mapping of asset adress => amount
114    mapping(address => uint256) public unreleasedPLPIncome;

    /// @notice The address of PLP contract
117    address public primeLiquidityProvider;

    /// @notice The address of ResilientOracle contract
120    ResilientOracleInterface public oracle;

    /// @dev This empty reserved space is put in place to allow future versions to add new
    /// variables without shifting down storage in the inheritance chain.
124    uint256[25] private __gap;

``` 
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L46


## [G-02] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

### Cache IVToken(market).underlying() outside of loop to save 1 STATICCALL per loop iteration

```solidity
file:   contracts/Tokens/Prime/Prime.sol

178     for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            uint256 interestAccrued = getInterestAccrued(market, user);
            uint256 accrued = interests[market][user].accrued;

            pendingInterests[i] = PendingInterest({
                market: IVToken(market).underlying(),
                amount: interestAccrued + accrued
            });

            unchecked {
                i++;
            }
        }

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L178-L191


## [G-03] Use assembly to perform efficient back-to-back calls

If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer), which can potentially allow us to avoid memory expansion costs. In this case, we are also able to efficiently store the function signatures together in memory as one word, saving multiple MLOADs in the process.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.


### use for this oracle.updateAssetPrice(xvsToken) back to bak  external call assembly

```solidity
file:   contracts/Tokens/Prime/Prime.sol

657     oracle.updateAssetPrice(xvsToken);
        oracle.updatePrice(market);


```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L657-L658

### use for this vToken.borrowBalanceStored() back to bak  external call assembly

```solidity
file:   contracts/Tokens/Prime/Prime.sol
 
496       function calculateAPR(address market, address user) external view returns (uint256 supplyAPR, uint256 borrowAPR) {
        IVToken vToken = IVToken(market);
        uint256 borrow = vToken.borrowBalanceStored(user);
        uint256 exchangeRate = vToken.exchangeRateStored();
        uint256 balanceOfAccount = vToken.balanceOf(user);
        uint256 supply = (exchangeRate * balanceOfAccount) / EXP_SCALE;

        uint256 userScore = interests[market][user].score;
        uint256 totalScore = markets[market].sumOfMembersScore;

        uint256 xvsBalanceForScore = _xvsBalanceForScore(_xvsBalanceOfUser(user));
        (, uint256 cappedSupply, uint256 cappedBorrow) = _capitalForScore(
            xvsBalanceForScore,
            borrow,
            supply,
            address(vToken)
        );

        return _calculateUserAPR(market, supply, borrow, cappedSupply, cappedBorrow, userScore, totalScore);
    }


```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496-L515


## [G-04] Use calldata instead of memory for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity
file:  Tokens/Prime/Prime.sol

200    function updateScores(address[] memory users) external {

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200

## [G-05] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking thefunction as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a paymentwas provided.


```solidity
file:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol

177  function setPrimeToken(address prime_) external onlyOwner {

216  function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177


## [G-06] Use hardcoded address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's script.sol and solmate's LibRlp.sol contracts can help achieve this.

```solidity
file:  contracts/Tokens/Prime/Prime.sol 

564    address(this),

682    if (amount > asset.balanceOf(address(this))) {

686    if (amount > asset.balanceOf(address(this))) {

960    address(this),


```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L564

```solidity
file:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol

217    uint256 balance = token_.balanceOf(address(this));

234    uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

259    uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));


```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L217

## [G-07] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.
see source:  
<https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27>


```solidity
file:  contracts/Tokens/Prime/libs/Scores.sol

52    bool lessxvsThanCapital = xvs < capital;


```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/Scores.sol#L52

```solidity
file:  contracts/Tokens/Prime/Prime.sol
 
222   isScoreUpdated[nextScoreUpdateRoundId][user] = true;

299   markets[vToken].exists = true;

707   tokens[user].exists = true;

750   tokens[user].exists = false;

751   tokens[user].isIrrevocable = false;

765   userToken.isIrrevocable = true;

909   return false;

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L222


```solidity
file:  contracts/Tokens/Prime/PrimeStorage.sol

8      bool exists;

9      bool isIrrevocable;

17     bool exists;

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L8

## [G-08] Expensive operation inside a for loop

```solidity
file:   contracts/Tokens/Prime/Prime.sol

178     for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            uint256 interestAccrued = getInterestAccrued(market, user);
            uint256 accrued = interests[market][user].accrued;

            pendingInterests[i] = PendingInterest({
                market: IVToken(market).underlying(),
                amount: interestAccrued + accrued
            });

            unchecked {
                i++;
            }
        }

204       for (uint256 i = 0; i < users.length; ) {
            address user = users[i];

            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

            address[] storage _allMarkets = allMarkets;
            for (uint256 j = 0; j < _allMarkets.length; ) {
                address market = _allMarkets[j];
                _executeBoost(user, market);
                _updateScore(user, market);

                unchecked {
                    j++;
                }
            }

            pendingScoreUpdates--;
            isScoreUpdated[nextScoreUpdateRoundId][user] = true;

            unchecked {
                i++;
            }

            emit UserScoreUpdated(user);
        }

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L178-L191

## [G-09] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.


```solidity
file:  contracts/Tokens/Prime/Prime.sol

453    if (msg.sender != protocolShareReserve) revert InvalidCaller();


```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L453

```solidity
file:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol

193     if (msg.sender != prime) revert InvalidCaller();

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L193

## [N-10] Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function

In development process, if you changed one of them you should find all of other to change and for large and complicatedprojects possible this change will be missed.

```solidity
file: contracts/Tokens/Prime/Prime.sol

/// @audit this revert is duplicated on line 105, 143, 144, 145, 146, 147, 148

104    revert InvalidAddress();

/// @audit this revert is duplicated on line 202

201    revert NoScoreUpdatesRequired();

/// @audit this revert is duplicated on line 726

207    revert UserHasNoPrimeToken()
 
/// @audit this revert is duplicated on line 457, 555

265    revert MarketNotSupported()

/// @audit this revert is duplicated on line 716, 769

318    revert InvalidLimit()

/// @audit this revert is duplicated on line 555, 265

457    revert MarketNotSupported()

/// @audit this revert is duplicated on line 398

705    revert IneligibleToClaim()

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L104


```solidity
file:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol

/// @audit this revert is duplicated on line 158, 346

100   revert InvalidArguments()

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L100

## [G-11] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

Note that in order to do this optimization safely we will need to cache and restore the free memory pointer after the loop. We will also set the zero slot (0x60) back to 0.

```solidity
file:   contracts/Tokens/Prime/Prime.sol
 
246      for (uint256 i = 0; i < allMarkets.length; ) {
            accrueInterest(allMarkets[i]);

            unchecked {
                i++;
            }
        }

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246-L252


```solidity
file:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol

119     for (uint256 i; i < tokens_.length; ) {
            _initializeToken(tokens_[i]);

            unchecked {
                ++i;
            }
        }

161   for (uint256 i; i < numTokens; ) {
            _ensureTokenInitialized(tokens_[i]);
            _setTokenDistributionSpeed(tokens_[i], distributionSpeeds_[i]);

            unchecked {
                ++i;
            }
        }

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L119-L124

## [G-12] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value

```solidity
file:  contracts/Tokens/Prime/Prime.sol

692    asset.safeTransfer(user, amount);

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L692

```solidity
file:   contracts/Tokens/Prime/PrimeLiquidityProvider.sol

224     token_.safeTransfer(to_, amount_);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L224

## [G-13] Should use arguments instead of state variable

state variables should not used in emit  ,  This will save near 97 gas  

```solidity
file:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol

180   emit PrimeTokenUpdated(prime, prime_);

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L180

## [G-14] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file:  contracts/Tokens/Prime/Prime.sol

179    address market = _allMarkets[i];

180    uint256 interestAccrued = getInterestAccrued(market, user);

181    uint256 accrued = interests[market][user].accrued;

205    address user = users[i];

210    address[] storage _allMarkets = allMarkets;

212    address market = _allMarkets[j];

336    Token storage userToken = tokens[users[i]];

626    address market = _allMarkets[i];

631    uint256 score = _calculateScore(market, account);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L179

## [G-15] Avoid contract existence checks by using low level calls  

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.   


```solidity
file:  contracts/Tokens/Prime/Prime.sol

292    bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);

656    address xvsToken = IXVSVault(xvsVault).xvsAddress();

685    IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);

878    address xvsToken = IXVSVault(xvsVault).xvsAddress();

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L292


```solidity
file:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol

204    IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);

234    uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

259    uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L204

## [G-16] Using delete statement can save gas

```solidity
file:   contracts/Tokens/Prime/PrimeLiquidityProvider.sol

200    tokenAmountAccrued[token_] = 0;

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L200

```solidity
file:   contracts/Tokens/Prime/Prime.sol

295     markets[vToken].rewardIndex = 0;

298     markets[vToken].sumOfMembersScore = 0;

376     stakedAt[user] = 0;

401     stakedAt[msg.sender] = 0;

677     interests[vToken][user].accrued = 0;

688     unreleasedPLPIncome[underlying] = 0;

736     interests[_allMarkets[i]][user].score = 0;

737     interests[_allMarkets[i]][user].rewardIndex = 0;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L295


## [G-17] Not using the named return variable when a function returns, wastes deployment gas
 
```solidity
file:    contracts/Tokens/Prime/Prime.sol

496     function calculateAPR(address market, address user) external view returns (uint256 supplyAPR, uint256 borrowAPR) {

533     ) external view returns (uint256 supplyAPR, uint256 borrowAPR) {

970     function _incomeDistributionYearly(address vToken) internal view returns (uint256 amount) {

1001    ) internal view returns (uint256 supplyAPR, uint256 borrowAPR) {

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496

## [G-18] Create IVToken,IPrimeLiquidityProvider and IERC20Upgradeable  variable  immutable variable to avoid redundant external calls

```solidity
file:   contracts/Tokens/Prime/Prime.sol

497    IVToken vToken = IVToken(market);

559    IPrimeLiquidityProvider _primeLiquidityProvider = IPrimeLiquidityProvider(primeLiquidityProvider);

680    IERC20Upgradeable asset = IERC20Upgradeable(underlying);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L497

## [G‑19] Inverting the condition of an if-else-statement wastes gas

Flipping the true and false blocks instead saves 3 gas

```solidity
file:    contracts/Tokens/Prime/PrimeLiquidityProvider.sol

264     uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L264

```solidity
file:   contracts/Tokens/Prime/Prime.sol

889    supply = supplyUSD > 0 ? (supply * supplyCapUSD) / supplyUSD : 0;

893    borrow = borrowUSD > 0 ? (borrow * borrowCapUSD) / borrowUSD : 0;

1012   supplyAPR = totalSupply == 0 ? 0 : ((userSupplyIncomeYearly * MAXIMUM_BPS) / totalSupply);

1013   borrowAPR = totalBorrow == 0 ? 0 : ((userBorrowIncomeYearly * MAXIMUM_BPS) / totalBorrow);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L889

```solidity 
file:   contracts/Tokens/Prime/libs/Scores.sol

55   int256 ratio = lessxvsThanCapital ? FixedMath.toFixed(xvs, capital) : FixedMath.toFixed(capital, xvs);

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/Scores.sol#L55


## [G-20] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file:  contracts/Tokens/Prime/Prime.sol

51    event Mint(address indexed user, bool isIrrevocable);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L51


```solidity
file:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol

33    event TokenDistributionSpeedUpdated(address indexed token, uint256 newSpeed);

36    event PrimeTokenUpdated(address oldPrimeToken, address newPrimeToken);

39    event TokensAccrued(address indexed token, uint256 amount);

42    event TokenTransferredToPrime(address indexed token, uint256 amount);

45    event SweepToken(address indexed token, address indexed to, uint256 sweepAmount);

48    event TokenInitialBalanceUpdated(address indexed token, uint256 balance);


```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L33


## [G-21] Don't initialize variables with default value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with itâ€™s default value costs unnecesary gas.


```solidity
file:  contracts/Tokens/Prime/Prime.sol

156    nextScoreUpdateRoundId = 0;

178    for (uint256 i = 0; i < _allMarkets.length;

204    for (uint256 i = 0; i < users.length;

211    for (uint256 j = 0; j < _allMarkets.length;

246    for (uint256 i = 0; i < allMarkets.length;

335    for (uint256 i = 0; i < users.length;

349    for (uint256 i = 0; i < users.length; ) {

376    stakedAt[user] = 0;

401    stakedAt[msg.sender] = 0;

460    unreleasedPSRIncome[_getUnderlying(address(market))] = 0;

609    for (uint256 i = 0; i < _allMarkets.length; ) {

625    for (uint256 i = 0; i < _allMarkets.length; ) {

677    interests[vToken][user].accrued = 0;

688    unreleasedPLPIncome[underlying] = 0;

730    for (uint256 i = 0; i < _allMarkets.length; ) {

736    interests[_allMarkets[i]][user].score = 0;

737    interests[_allMarkets[i]][user].rewardIndex = 0;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L156

```solidity
file:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol

200    tokenAmountAccrued[token_] = 0;

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L200

```solidity
file:   contracts/Tokens/Prime/libs/FixedMath0x.sol

46     int256 private constant EXP_MAX_VAL = 0;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L46


## [G-22] Duplicated require()/if() checks should be refactored to a modifier or function   


to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file:  contracts/Tokens/Prime/Prime.sol

/// @audit this if is duplicated on line 726

207    if (!tokens[user].exists)

/// @audit this if is duplicated on line 710

334    if (isIrrevocable)

/// @audit this if is duplicated on line 744

370    if (tokens[user].isIrrevocable)


/// @audit this if is duplicated on line 686

682    if (amount > asset.balanceOf(address(this)))

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L207


```solidity
file:   contracts/Tokens/Prime/PrimeLiquidityProvider.sol

/// @audit this if is duplicated on line 157

99    if (numTokens != distributionSpeeds_.length)

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L99

```solidity
file:  contracts/Tokens/Prime/libs/FixedMath.sol

/// @audit this if is duplicated on line 47

35    if (f < 0)

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L35


## [G-23] for mapping use private instead of public if they mapping is used inside they contract 

berfore using private gas cost //152982
after using of priavate gas cost // 77126

tools used Remix

```solidity
file: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

21    mapping(address => uint256) public tokenDistributionSpeeds;

24    mapping(address => uint256) public lastAccruedBlock;

27    mapping(address => uint256) public tokenAmountAccrued;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L21

## [G-24] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time the variable is used, which wastes some gas.

```solidity
file:  contracts/Tokens/Prime/PrimeStorage.sol

34    uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;

37    uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

40    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L34

## [G-25] Loop best practice to save gas


```solidity
function Plusi() public view {
		for(uint i=0; i<10;) {
			console.log("Number ==",i);
			unchecked{
				++i;
			}
		}
	}
best practice
-----------------------------------------------------
for (uint i = 0; i < length; i = unchecked_inc(i)) {
    // do something that doesn't change the value of i
}
function unchecked_inc(uint i) returns (uint) {
    unchecked {
        return i + 1;
    }
}

```

```solidity
file:  contracts/Tokens/Prime/Prime.sol

188   unchecked {
                i++;
            }

216   unchecked {
                    j++;
                }

249    unchecked {
                i++;
            }

344   unchecked {
                i++;
            }

354   unchecked {
                    i++;
                }
613   unchecked {
                i++;
            }

635   unchecked {
                i++;
            }

739   unchecked {
                i++;
            }

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L188-L190

```solidity
file:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol

107   unchecked {
                ++i;
            }

122   unchecked {
                ++i;
            }

165   unchecked {
                ++i;
            }

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L107-L109

## [G-26] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

```solidity
file:  contracts/Tokens/Prime/Prime.sol

176    PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

683    address[] memory assets = new address[](1);

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L176


## [G‑27] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity
file:   contracts/Tokens/Prime/Prime.sol

246    for (uint256 i = 0; i < allMarkets.length; ) {
            accrueInterest(allMarkets[i]);

            unchecked {
                i++;
            }
        }

349    for (uint256 i = 0; i < users.length; ) {
                _mint(false, users[i]);
                _initializeMarkets(users[i]);
                delete stakedAt[users[i]];

                unchecked {
                    i++;
                }
            }

609   for (uint256 i = 0; i < _allMarkets.length; ) {
            _executeBoost(user, _allMarkets[i]);
            _updateScore(user, _allMarkets[i]);

            unchecked {
                i++;
            }
        }

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246-L252

```solidity
file:  contracts/Tokens/Prime/PrimeLiquidityProvider.sol

103   for (uint256 i; i < numTokens; ) {
            _initializeToken(tokens_[i]);
            _setTokenDistributionSpeed(tokens_[i], distributionSpeeds_[i]);

            unchecked {
                ++i;
            }
        }

119    for (uint256 i; i < tokens_.length; ) {
            _initializeToken(tokens_[i]);

            unchecked {
                ++i;
            }
        }

161    for (uint256 i; i < numTokens; ) {
            _ensureTokenInitialized(tokens_[i]);
            _setTokenDistributionSpeed(tokens_[i], distributionSpeeds_[i]);

            unchecked {
                ++i;
            }
        }

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L103-L111

## [G-28] State variables can be packed into fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions, we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).


```solidity
file:  contracts/Tokens/Prime/PrimeStorage.sol

70  address[] internal allMarkets;

    /// @notice numberator of alpha. Ex: if alpha is 0.5 then this will be 1
    uint128 public alphaNumerator;

    /// @notice denominator of alpha. Ex: if alpha is 0.5 then this will be 2
    uint128 public alphaDenominator;

    /// @notice address of XVS vault
    address internal xvsVault;

    /// @notice address of XVS vault reward token
    address internal xvsVaultRewardToken;

    /// @notice address of XVS vault pool id
    uint256 internal xvsVaultPoolId;

    /// @notice mapping to check if a account's score was updated in the round
    mapping(uint256 => mapping(address => bool)) public isScoreUpdated;

    /// @notice unique id for next round
    uint256 public nextScoreUpdateRoundId;

    /// @notice total number of accounts whose score needs to be updated
    uint256 public totalScoreUpdatesRequired;

    /// @notice total number of accounts whose score is yet to be updated
    uint256 public pendingScoreUpdates;

    /// @notice mapping used to find if an asset is part of prime markets
    mapping(address => address) public vTokenForAsset;

    /// @notice address of protocol share reserve contract
    address public protocolShareReserve;

    /// @notice address of core pool comptroller contract
    address public comptroller;

    /// @notice unreleased income from PSR that's already distributed to prime holders
    /// @dev mapping of asset adress => amount
    mapping(address => uint256) public unreleasedPSRIncome;

    /// @notice unreleased income from PLP that's already distributed to prime holders
    /// @dev mapping of asset adress => amount
    mapping(address => uint256) public unreleasedPLPIncome;

    /// @notice The address of PLP contract
    address public primeLiquidityProvider;

    /// @notice The address of ResilientOracle contract
    ResilientOracleInterface public oracle;

    /// @dev This empty reserved space is put in place to allow future versions to add new
    /// variables without shifting down storage in the inheritance chain.
    uint256[25] private __gap;

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L70-L124

## [G-29] Move storage pointer to top of function to avoid offset calculation

We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.


```solidity 
file:  contracts/Tokens/Prime/Prime.sol
 
336   Token storage userToken = tokens[users[i]];

```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L336