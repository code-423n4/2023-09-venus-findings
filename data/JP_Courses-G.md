The below demonstrate some gas optimizations/savings opportunities in the Prime.sol contract:

++ indicates added lines/code with improvement/optimizations.
-- indicates replaced lines

0. Prime::getPendingInterests():

https://github.com/code-423n4/2023-09-venus/blob/23f5db740d8a794ac563ac32195b675c53042bb4/contracts/Tokens/Prime/Prime.sol#L174-L194

```solidity
    function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
        address[] storage _allMarkets = allMarkets;
    ++  uint256 _allMarketsLength = _allMarkets.length;
    --  PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);
    ++  PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarketsLength);

    --  for (uint256 i = 0; i < _allMarkets.length; ) {
    ++  for (uint256 i; i < _allMarketsLength; ) {
```
From above:

`++ uint256 _allMarketsLength = _allMarkets.length;`: 
This change saves gas by caching the length of `_allMarkets` in a separate variable `_allMarketsLength` outside of the loop, reducing the need to repeatedly access the length of the array from storage.
   
`-- PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);`: 
The length of the `pendingInterests` array is now set to the memory cached `_allMarketsLength` for gas savings.
   
`++ for (uint256 i; i < _allMarketsLength; ) {`:
Saving some gas by removing the redundant explicit initialization for `i`.


(similar explanations for some of the below gas findings.)

1. Prime::updateScores():

https://github.com/code-423n4/2023-09-venus/blob/23f5db740d8a794ac563ac32195b675c53042bb4/contracts/Tokens/Prime/Prime.sol#L200-L230

```solidity
    function updateScores(address[] memory users) external {
        if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
        if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

        for (uint256 i = 0; i < users.length; ) {
            address user = users[i];

            if (!tokens[user].exists) revert UserHasNoPrimeToken();
            if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

            address[] storage _allMarkets = allMarkets;
        ++  uint256 _allMarketsLength = _allMarkets.length;
        --  for (uint256 j = 0; j < _allMarkets.length; ) {
        ++  for (uint256 j; j < _allMarketsLength; ) {
```

2. Prime::updateAlpha():

https://github.com/code-423n4/2023-09-venus/blob/23f5db740d8a794ac563ac32195b675c53042bb4/contracts/Tokens/Prime/Prime.sol#L237-L255

```solidity
    function updateAlpha(uint128 _alphaNumerator, uint128 _alphaDenominator) external {
        _checkAccessAllowed("updateAlpha(uint128,uint128)");
        _checkAlphaArguments(_alphaNumerator, _alphaDenominator);

        emit AlphaUpdated(alphaNumerator, alphaDenominator, _alphaNumerator, _alphaDenominator);

        alphaNumerator = _alphaNumerator;
        alphaDenominator = _alphaDenominator;
        
    ++  uint256 allMarketsLength = allMarkets.length;
    --  for (uint256 i = 0; i < allMarkets.length; ) {
    ++  for (uint256 i; i < allMarketsLength; ) {
```

3. Prime::addMarket():

https://github.com/code-423n4/2023-09-venus/blob/23f5db740d8a794ac563ac32195b675c53042bb4/contracts/Tokens/Prime/Prime.sol#L288-L309

```solidity
    function addMarket(address vToken, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
        _checkAccessAllowed("addMarket(address,uint256,uint256)");
        if (markets[vToken].exists) revert MarketAlreadyExists();

        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);
        if (!isMarketExist) revert InvalidVToken();

        markets[vToken].rewardIndex = 0;
        markets[vToken].supplyMultiplier = supplyMultiplier;
        markets[vToken].borrowMultiplier = borrowMultiplier;
        markets[vToken].sumOfMembersScore = 0;
        markets[vToken].exists = true;

        vTokenForAsset[_getUnderlying(vToken)] = vToken;

        allMarkets.push(vToken);
        _startScoreUpdateRound();

    ++	uint256 allMarketsLength = allMarkets.length;
    --  _ensureMaxLoops(allMarkets.length);
    ++  _ensureMaxLoops(allMarketsLength);

        emit MarketAdded(vToken, supplyMultiplier, borrowMultiplier);
    }
```

4. Prime::_accrueInterestAndUpdateScore():

https://github.com/code-423n4/2023-09-venus/blob/23f5db740d8a794ac563ac32195b675c53042bb4/contracts/Tokens/Prime/Prime.sol#L607-L617

```solidity
    function _accrueInterestAndUpdateScore(address user) internal {
        address[] storage _allMarkets = allMarkets;
    ++  uint256 _allMarketsLength; = _allMarkets.length;
    --  for (uint256 i = 0; i < _allMarkets.length; ) {
    ++  for (uint256 i; i < _allMarketsLength; ) {
```

5. Prime::_initializeMarkets():

https://github.com/code-423n4/2023-09-venus/blob/23f5db740d8a794ac563ac32195b675c53042bb4/contracts/Tokens/Prime/Prime.sol#L623-L639

```solidity
    function _initializeMarkets(address account) internal { 
        address[] storage _allMarkets = allMarkets;
    ++  uint256 _allMarketsLength = _allMarkets.length;
    --  for (uint256 i = 0; i < _allMarkets.length; ) {
    ++  for (uint256 i; i < _allMarketsLength; ) {
```

6. Prime::_burn():

https://github.com/code-423n4/2023-09-venus/blob/23f5db740d8a794ac563ac32195b675c53042bb4/contracts/Tokens/Prime/Prime.sol#L725-L742

```solidity
    function _burn(address user) internal {
        if (!tokens[user].exists) revert UserHasNoPrimeToken();

        address[] storage _allMarkets = allMarkets;

    ++  uint256 _allMarketsLength = _allMarkets.length;
    --  for (uint256 i = 0; i < _allMarkets.length; ) {
    ++  for (uint256 i; i < _allMarketsLength; ) {
```

7. Prime::_capitalForScore():

https://github.com/code-423n4/2023-09-venus/blob/23f5db740d8a794ac563ac32195b675c53042bb4/contracts/Tokens/Prime/Prime.sol#L872-L897

```solidity
    function _capitalForScore(
        uint256 xvs,
        uint256 borrow,
        uint256 supply,
        address market
    ) internal view returns (uint256, uint256, uint256) {
        address xvsToken = IXVSVault(xvsVault).xvsAddress();

        uint256 xvsPrice = oracle.getPrice(xvsToken);
    ++  uint256 _marketBorrowMultiplier = markets[market].borrowMultiplier;
    ++  uint256 _marketSupplyMultiplier = markets[market].supplyMultiplier;
    --  uint256 borrowCapUSD = (xvsPrice * ((xvs * markets[market].borrowMultiplier) / EXP_SCALE)) / EXP_SCALE;
    --  uint256 supplyCapUSD = (xvsPrice * ((xvs * markets[market].supplyMultiplier) / EXP_SCALE)) / EXP_SCALE;
    ++  uint256 borrowCapUSD = (xvsPrice * ((xvs * _marketBorrowMultiplier) / EXP_SCALE)) / EXP_SCALE;
    ++  uint256 supplyCapUSD = (xvsPrice * ((xvs * _marketSupplyMultiplier) / EXP_SCALE)) / EXP_SCALE;
```
From above:

`++ uint256 _marketBorrowMultiplier = markets[market].borrowMultiplier;` and 
`++ uint256 _marketSupplyMultiplier = markets[market].supplyMultiplier;`: 
These lines cache the `borrowMultiplier` and `supplyMultiplier` from the `markets` mapping in separate variables, optimizing gas usage by avoiding repeated mapping access from storage during calculations.

`-- uint256 borrowCapUSD = (xvsPrice * ((xvs * markets[market].borrowMultiplier) / EXP_SCALE)) / EXP_SCALE;` and 
`-- uint256 supplyCapUSD = (xvsPrice * ((xvs * markets[market].supplyMultiplier) / EXP_SCALE)) / EXP_SCALE;`: 
These lines replace mapping access from storage with the memory cached `_marketBorrowMultiplier` and `_marketSupplyMultiplier`, respectively, saving gas.
