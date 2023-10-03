## A. Reordering Operations in the `setPrimeToken` Function:
[Link](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177-L182)
The `setPrimeToken` function allows the owner of the contract to update the address of the prime token contract and emits an event to notify observers of this change. However, the problem lies in the ordering of operations within the function:
```solidity
_emit PrimeTokenUpdated(prime, prime_);
prime = prime_;
```
Here, the event `PrimeTokenUpdated` is emitted before updating the `prime` state variable with the new address `prime_`. This sequence means that external contracts or observers listening to this event will receive the old value of prime, not the updated one. It can lead to external code relying on outdated information, potentially causing unexpected behavior.
## Impact:
The impact of this issue is that external code relying on the emitted event may make decisions or take actions based on outdated information about the address of the prime token contract. This could result in errors, inconsistencies, or vulnerabilities in the behavior of the contract or interacting contracts.
## Mitigation:
To address this issue, the order of operations in the `setPrimeToken` function should be corrected. The state variable `prime` should be updated before emitting the event, as follows:
```solidity
function setPrimeToken(address prime_) external onlyOwner {
    _ensureZeroAddress(prime_);

    prime = prime_;  // Update the state variable first
    emit PrimeTokenUpdated(prime, prime_); // Then emit the event
}
```
## B. Gas Limit Challenge in Processing Large Arrays: 
[Link](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L174-L194)
The vulnerability arises in the provided Solidity function `getPendingInterests`, which is designed to calculate pending interest accrued for a user across all markets. The vulnerability details are as follows:
```solidity
function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
    address[] storage _allMarkets = allMarkets;
    PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

    for (uint256 i = 0; i < _allMarkets.length; ) {
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

    return pendingInterests;
}

```
The function loops through the `_allMarkets` array to calculate pending interest for a user across all markets. The gas limit challenge arises if `_allMarkets` contains a large number of elements. In such cases, the gas cost of processing the loop may exceed the gas limit set for Ethereum transactions.

##  Impact:
The impact of this gas limit challenge is that transactions invoking the getPendingInterests function may fail when processing a large number of markets. Users will not receive the expected results, and the contract's functionality may be limited when dealing with extensive data sets.
## Mitigation: 
To mitigate the gas limit challenge when processing large arrays, consider implementing batch processing. 