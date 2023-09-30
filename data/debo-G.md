### [G-01] Don't Initialize Variables with Default Value
## Impact
I'll focus on the getPendingInterests function and the initialisation of variables within it.

In the getPendingInterests function, the variable i is initialised to 0 in the for loop. This is necessary and not a gas optimisation issue because i is used as the index to iterate over _allMarkets.

However, there is a potential gas optimisation issue with the initialisation of the pendingInterests array. The array is initialised with a length equal to _allMarkets.length, which means that memory is allocated for each element in the array, even if not all elements are used. This could potentially waste gas if _allMarkets.length is large and not all elements are used.

A potential optimisation could be to use a dynamic array and push elements onto the array as needed. However, this would also increase gas costs for each push operation, so the optimal approach would depend on the expected size of _allMarkets and the number of elements expected to be used.

In terms of security, there doesn't seem to be any direct vulnerabilities related to variable initialisation in this function. However, the function does not check if market is a valid address before using it to call getInterestAccrued and access interests[market][user]. If market is not a valid address, these operations could potentially fail.

Here's a potential PoC for an exploit:
```sol
// Assume attacker has control over the allMarkets array
address[] allMarkets = [address(0)];

// Attacker calls getPendingInterests
prime.getPendingInterests(attacker);
```
In this case, the call to getInterestAccrued would fail because address(0) is not a valid contract address. This could potentially be used to cause the getPendingInterests function to revert, denying service to legitimate users. However, this would require the attacker to have control over the allMarkets array, which is unlikely in a properly secured contract.
## References
```sol
2023-09-venus/contracts/Tokens/Prime/Prime.sol::178 => for (uint256 i = 0; i < _allMarkets.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::204 => for (uint256 i = 0; i < users.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::211 => for (uint256 j = 0; j < _allMarkets.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::246 => for (uint256 i = 0; i < allMarkets.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::335 => for (uint256 i = 0; i < users.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::349 => for (uint256 i = 0; i < users.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::609 => for (uint256 i = 0; i < _allMarkets.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::625 => for (uint256 i = 0; i < _allMarkets.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::730 => for (uint256 i = 0; i < _allMarkets.length; ) {
```
