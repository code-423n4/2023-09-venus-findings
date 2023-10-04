# Gas Optimization

# Summary

| Number | Gas Optimization                                                                           | Context |
| :----: | :----------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Functions Guaranteed to revert when called by normal users can be markd payable            |    3    |
| [G-02] | Do not calculate constants                                                                 |    3    |
| [G-03] | Unnecessary casting as variable is already of the same type                                |    2    |
| [G-04] | Revert() statements should be used sorted from cheapest to most expensive                  |    6    |
| [G-05] | Make the variable outside the loop to save gas                                             |    4    |
| [G-06] | Using delete statement can save gas                                                        |   10    |
| [G-07] | Using Storage instead of memory for structs/arrays saves gas                               |    2    |
| [G-08] | Amounts should be checked for 0 before calling a transfer                                  |    1    |
| [G-09] | Use ++i instead of i++ to increment                                                        |   14    |
| [G-10] | Should use arguments instead of the state variables                                        |    3    |
| [G-11] | Use assembly to perform efficient back-to-back calls                                       |   17    |
| [G-12] | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GASReturns |   21    |
| [G-13] | Ternary operation is cheaper than if-else statement                                        |    4    |
| [G-14] | Use hardcode address instead address(this)                                                 |    7    |
| [G-15] | Use do while loops instead of for loops                                                    |   12    |
| [G-16] | Sort Solidity operations using short-circuit mode                                          |   10    |
| [G-17] | Use uint256(1)/uint256(2 instead for true and false boolean states                         |    8    |

## [G-01] Functions Guaranteed to revert when called by normal users can be markd payable

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

118    function initializeTokens(address[] calldata tokens_) external onlyOwner {


177    function setPrimeToken(address prime_) external onlyOwner {


    216    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

## [G-02] Do not calculate constants

```solidity
File: contracts/Tokens/Prime/PrimeStorage.sol

34     uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;

37     uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

40    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L34

## [G-03] Unnecessary casting as variable is already of the same type

```solidity
File: contracts/Tokens/Prime/Prime.sol

511            address(vToken)

684            assets[0] = address(asset);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L511

## [G-04] Revert() statements should be used sorted from cheapest to most expensive

```solidity
File: contracts/Tokens/Prime/Prime.sol

143        if (_xvsVault == address(0)) revert InvalidAddress();
144        if (_xvsVaultRewardToken == address(0)) revert InvalidAddress();
145        if (_protocolShareReserve == address(0)) revert InvalidAddress();
146        if (_comptroller == address(0)) revert InvalidAddress();
147        if (_oracle == address(0)) revert InvalidAddress();
148        if (_primeLiquidityProvider == address(0)) revert InvalidAddress();


456        address vToken = vTokenForAsset[asset];
457        if (vToken == address(0)) revert MarketNotSupported();
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

## [G-05] Make the variable outside the loop to save gas

When you declare a variable inside a loop the variable is reinitialized and assigned memory or storage space during each iteration of the loop. This can use more gas, leading to higher transaction costs. To save gas, it's often recommended to declare the variable outside the loop so that it is only initialized once and reused throughout the loop.

```solidity
File: contracts/Tokens/Prime/Prime.sol

179            address market = _allMarkets[i];
180            uint256 interestAccrued = getInterestAccrued(market, user);
181            uint256 accrued = interests[market][user].accrued;


205            address user = users[i];

212                address market = _allMarkets[j];

626            address market = _allMarkets[i];

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

## [G-06] Using delete statement can save gas

```solidity
File: contracts/Tokens/Prime/Prime.sol

156        nextScoreUpdateRoundId = 0;

298        markets[vToken].sumOfMembersScore = 0;

376            stakedAt[user] = 0;

401        stakedAt[msg.sender] = 0;

460        unreleasedPSRIncome[_getUnderlying(address(market))] = 0;

677        interests[vToken][user].accrued = 0;

688                unreleasedPLPIncome[underlying] = 0;

736            interests[_allMarkets[i]][user].score = 0;

737            interests[_allMarkets[i]][user].rewardIndex = 0;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L156

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

200        tokenAmountAccrued[token_] = 0;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L200

## [G-07] Using Storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to
be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File: contracts/Tokens/Prime/Prime.sol

176        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

683            address[] memory assets = new address[](1);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L176

## [G-08] Amounts should be checked for 0 before calling a transfer

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

204        IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L204

## [G-09] Use ++i instead of i++ to increment

```solidity
File: contracts/Tokens/Prime/Prime.sol

189                i++;

217                    j++;

225                i++;

250                i++;

345                    i++;

355                    i++;

614                i++;

636                i++;

711            totalIrrevocable++;

713            totalRevocable++;

740                i++;

766        totalIrrevocable++;

819        nextScoreUpdateRoundId++;

828        if (totalScoreUpdatesRequired > 0) totalScoreUpdatesRequired--;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

## [G-10] Should use arguments instead of the state variables

```solidity
File: contracts/Tokens/Prime/Prime.sol

241        emit AlphaUpdated(alphaNumerator, alphaDenominator, _alphaNumerator, _alphaDenominator);


462        emit UpdatedAssetsState(comptroller, asset);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

180        emit PrimeTokenUpdated(prime, prime_);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

## [G-11] Use assembly to perform efficient back-to-back calls

```solidity
File: contracts/Tokens/Prime/Prime.sol

498        uint256 borrow = vToken.borrowBalanceStored(user);
499        uint256 exchangeRate = vToken.exchangeRateStored();
500        uint256 balanceOfAccount = vToken.balanceOf(user);


561                uint256 totalIncomeUnreleased = IProtocolShareReserve(protocolShareReserve).getUnreleasedFunds(


570        _primeLiquidityProvider.accrueTokens(underlying);
571        uint256 totalAccruedInPLP = _primeLiquidityProvider.tokenAmountAccrued(underlying);





651        uint256 borrow = vToken.borrowBalanceStored(user);
652        uint256 exchangeRate = vToken.exchangeRateStored();
653        uint256 balanceOfAccount = vToken.balanceOf(user);


656        address xvsToken = IXVSVault(xvsVault).xvsAddress();
657        oracle.updateAssetPrice(xvsToken);
658        oracle.updatePrice(market);




685            IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
686            if (amount > asset.balanceOf(address(this))) {
687                IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));





880        uint256 xvsPrice = oracle.getPrice(xvsToken);

884        uint256 tokenPrice = oracle.getUnderlyingPrice(market);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L498-L500

## [G-12] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GASReturns

The solidity compiler outputs more efficient code when the variable is declared in the return statement. There seem to be very few exceptions to this in practice, so if you see an anonymous return, you should test it with a named return instead to determine which case is most efficient.

```solidity
File: contracts/Tokens/Prime/Prime.sol

193        return pendingInterests;

434        return _claimInterest(vToken, msg.sender);

444        return _claimInterest(vToken, user);

470        return allMarkets;

479        if (stakedAt[user] == 0) return STAKING_PERIOD;

514        return _calculateUserAPR(market, supply, borrow, cappedSupply, cappedBorrow, userScore, totalScore);

547        return _calculateUserAPR(market, supply, borrow, cappedSupply, cappedBorrow, userScore, totalScore);

600        return _interestAccrued(vToken, user);

663        return Scores.calculateScore(xvsBalanceForScore, capital, alphaNumerator, alphaDenominator);

696        return amount;

846        return (xvs - pendingWithdrawals);

896        return ((supply + borrow), supply, borrow);

922        return (index * score) / EXP_SCALE;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

```solidity
File: Tokens/Prime/PrimeLiquidityProvider.sol

238            return distributionSpeed;

277        return block.number;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

```solidity
File: contracts/Tokens/Prime/libs/FixedMath.sol

25        return (n.toInt256() * FixedMath0x.FIXED_1) / int256(d.toInt256());

37        return uint256((u.toInt256() * FixedMath0x.FIXED_1) / f);

49        return uint256((u.toInt256() * f) / FixedMath0x.FIXED_1);

54        return FixedMath0x.ln(x);

59        return FixedMath0x.exp(x);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol

## [G-13] Ternary operation is cheaper than if-else statement

```solidity
File: contracts/Tokens/Prime/Prime.sol


421        if (paused()) {
422            _unpause();
423        } else {
424            _pause();
425        }




482        if (totalTimeStaked < STAKING_PERIOD) {
483            return STAKING_PERIOD - totalTimeStaked;
484        } else {
485            return 0;
486        }



855        if (xvs > MAXIMUM_XVS_CAP) {
856            return MAXIMUM_XVS_CAP;
857        } else {
858            return xvs;
859        }




931        if (vToken == VBNB) {
932            return WBNB;
933        } else {
934            return IVToken(vToken).underlying();
935        }
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L931-L935

## [G-14] Use hardcode address instead address(this)

```solidity
File: contracts/Tokens/Prime/Prime.sol

564            address(this),

682        if (amount > asset.balanceOf(address(this))) {

686            if (amount > asset.balanceOf(address(this))) {

960                address(this),

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol


217        uint256 balance = token_.balanceOf(address(this));

234        uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

259            uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

## [G-15] Use do while loops instead of for loops

```solidity
File: contracts/Tokens/Prime/Prime.sol

178        for (uint256 i = 0; i < _allMarkets.length; ) {

204        for (uint256 i = 0; i < users.length; ) {

211            for (uint256 j = 0; j < _allMarkets.length; ) {

246        for (uint256 i = 0; i < allMarkets.length; ) {

335            for (uint256 i = 0; i < users.length; ) {

349            for (uint256 i = 0; i < users.length; ) {

609        for (uint256 i = 0; i < _allMarkets.length; ) {

625        for (uint256 i = 0; i < _allMarkets.length; ) {

730        for (uint256 i = 0; i < _allMarkets.length; ) {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L204

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

103        for (uint256 i; i < numTokens; ) {

119        for (uint256 i; i < tokens_.length; ) {

161        for (uint256 i; i < numTokens; ) {

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L161

## [G-16] Sort Solidity operations using short-circuit mode

```solidity
File: contracts/Tokens/Prime/Prime.sol

318        if (_irrevocableLimit < totalIrrevocable || _revocableLimit < totalRevocable) revert InvalidLimit();


337                if (userToken.exists && !userToken.isIrrevocable) {

369        if (tokens[user].exists && !isAccountEligible) {

375        } else if (!isAccountEligible && !tokens[user].exists && stakedAt[user] > 0) {

377        } else if (stakedAt[user] == 0 && isAccountEligible && !tokens[user].exists) {

379        } else if (tokens[user].exists && isAccountEligible) {

780        if (!markets[vToken].exists || !tokens[user].exists) {

795        if (!markets[market].exists || !tokens[user].exists) {

810        if (_alphaDenominator == 0 || _alphaNumerator > _alphaDenominator) {

830        if (pendingScoreUpdates > 0 && !isScoreUpdated[nextScoreUpdateRoundId][user]) {

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L318

## [G-17] Use uint256(1)/uint256(2 instead for true and false boolean states

```solidity
File: contracts/Tokens/Prime/Prime.sol

292        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);

331    function issue(bool isIrrevocable, address[] calldata users) external {

367        bool isAccountEligible = isEligible(totalStaked);

704    function _mint(bool isIrrevocable, address user) internal {

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L292

```solidity
File:  contracts/Tokens/Prime/PrimeStorage.sol

8        bool exists;
9        bool isIrrevocable;

17        bool exists;

88    mapping(uint256 => mapping(address => bool)) public isScoreUpdated;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L8