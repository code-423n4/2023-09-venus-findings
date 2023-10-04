ID|Title
-|-
L-01|Missing check for market existence before APR calculation
L-02|Shadowing named return variable declaration
L-03|Excessive rights for claimInterest()
L-04|The calculation of `userYearlyIncome` uses the number of blocks mined annually
L-05|Incomplete configuration of _setMaxLoopsLimit()
N-01|Incorrect comments
N-02|Incorrect gap size
N-03|Typos

## [L-01] Missing check for market existence before APR calculation

An explicit check for `market` existence could be added to the `calculateAPR()` and `estimateAPR()` functions, so that when a user inputs an unsupported market, the function will throw appropriate error messages.

[Prime.sol#L496-L497](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L496-L497)
```js
function calculateAPR(address market, address user) external view returns (uint256 supplyAPR, uint256 borrowAPR) {
    IVToken vToken = IVToken(market);
```

[Prime.sol#L527-L534](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L527-L534)
```js
function estimateAPR(
    ...
) external view returns (uint256 supplyAPR, uint256 borrowAPR) {
    uint256 totalScore = markets[market].sumOfMembersScore - interests[market][user].score;
```

## [L-02] Shadowing named return variable declaration

*1 instance*

[Prime.sol#L174-L176](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L174-L176)
```diff
function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
    address[] storage _allMarkets = allMarkets;
-   PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);
+   pendingInterests = new PendingInterest[](_allMarkets.length);
```

## [L-03] Excessive rights for claimInterest()

Function `claimInterest()` in `Prime.sol` allows anyone to claim any user's interest.

```js
function claimInterest(address vToken, address user) external whenNotPaused returns (uint256) {
    return _claimInterest(vToken, user);
}
```

In some cases, user do not want to receive funds right now (leak of keys and being in process in negotiation with validators etc.)

## [L-04] The calculation of `userYearlyIncome` uses the number of blocks mined annually

The calculation of userYearlyIncome uses the number of blocks mined annually. The code will be deployed on BNB Chain, Ethereum mainnet, Arbitrum, Polygon zkEVM and opBNB. It'll be fine for BNB Chain and Ethereum mainnet, which have a fixed block produce time. However, on Arbitrum, Polygon zkEVM and opBNB, BLOCKS_PER_YEAR is an estimated value.

## [L-05] Incomplete configuration of _setMaxLoopsLimit()

Unavailable to add enough markets if `maxLoopsLimit` of `MaxLoopsLimitHelper()` is set too small, and there's no external function to update `maxLoopsLimit`.

Abstract contract `MaxLoopsLimitHelper` is used to control length of array in contracts. This limit is setting during initialize of SC's. There is loose requirement which control this length exactly at init process (0 by default).

```js
require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");
```

## [N-01] Incorrect comments

According to the [document](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/README.md#income-collection-and-distribution) and implementation, `_executeBoost` is called by Comptroller *after* changing account's borrow or supply balance, while it's stated *before* in the comment.

> In Comptroller (specifically in the `PolicyFacet`), **after** executing any operation that could impact the Prime score or interest, we accrue the interest and update the score for the Prime user by calling `accrueInterestAndUpdateScore`.

[Prime.sol#L775](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L775)
```js
/**
 * @notice Accrue rewards for the user. Must be called by Comptroller before changing account's borrow or supply balance.
 * @param user account for which we need to accrue rewards
 * @param vToken the market for which we need to accrue rewards
 */
function _executeBoost(address user, address vToken) internal {
```

## [N-02] Incorrect gap size

In the PrimeStorageV1 contract, the number of used storage slots is 24, while the gap size is 25.

[PrimeStorage.sol#L124](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L124)

## [N-03] Typos

*2 instances*

- *interes* should be *interest*.

    [Prime.sol#385](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L385C24-L385C31)
    ```js
    * @notice accrues interes and updates score for an user for a specific market
    ```

- *intialized* should be *initialized*.

    [PrimeLiquidityProvider.sol#115](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L115C62-L115C72)
    ```js
    * @param tokens_ Array of addresses of the tokens to be intialized
    ```