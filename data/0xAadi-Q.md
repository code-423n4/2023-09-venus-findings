## Summary

### Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [[L&#x2011;01](#l01-missing-events)] | Missing Event for critical functions or parameters change | 4 | 

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-typos)] | Typos | 5 | 


## Low Risk Issues


### [L&#x2011;01] Missing Event for critical functions or parameters change

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

*There is 4 instance of this issue:*

```solidity
File: contracts/Tokens/Prime/Prime.sol

331:    function issue(bool isIrrevocable, address[] calldata users) external {
332:        _checkAccessAllowed("issue(bool,address[])");

365:    function xvsUpdated(address user) external {
366:        uint256 totalStaked = _xvsBalanceOfUser(user);

389:    function accrueInterestAndUpdateScore(address user, address market) external {
390:        _executeBoost(user, market);

397:    function claim() external {
398:        if (stakedAt[msg.sender] == 0) revert IneligibleToClaim();

```
*GitHub*: [331](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L331C1-L332C54),[365](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L365C1-L366C55),[389](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L389C1-L390C37),[397](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L397C1-L398C67)

## Non-critical Issues

### [N&#x2011;01] Typos

*There are 5 instances of this issue:*

```solidity
File: contracts/Tokens/Prime/Prime.sol

// @audit re-initializers
111: // Note that the contract is upgradeable. Use initialize() or reinitializers

// @audit interest
385: * @notice accrues interes and updates score for an user for a specific market

// @audit interest
604: * @notice accrues interes and updates score of all markets for an user

```
*GitHub*: [111](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L111), [385](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L385),[604](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L604)

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

// @audit initialized
115: * @param tokens_ Array of addresses of the tokens to be intialized

// @audit initialized
282: * @param token_ Address of the token to be intialized

```
*GitHub*: [115](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L115),[282](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L282)


