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
### [G-02] Cache Array Length Outside of Loop
## Impact
**Exploit Scenario** The code does not cache the length of the array outside of the loop. 
This means that every time the loop iterates, it has to access the length of the array from storage. 
This is a costly operation in terms of gas usage. 
An attacker could potentially exploit this by sending transactions with large arrays, causing the contract to consume more gas than necessary.

**Impact** The impact of this issue is primarily economic. 
It does not pose a direct security risk, but it can lead to higher gas costs for transactions interacting with the contract. 
This could potentially make the contract less attractive to users due to the increased costs of interaction.

**Recommendation** To optimise gas usage, the length of the array should be cached outside of the loop. 
This can be done by declaring a variable before the loop starts and assigning it the length of the array. 
This way, the length of the array is only accessed once from storage, significantly reducing gas costs.

For example, the loop in the initialise function could be optimised as follows:
```sol
uint256 numTokens = tokens_.length;
for (uint256 i = 0; i < numTokens; i++) {
    _initializeToken(tokens_[i]);
    _setTokenDistributionSpeed(tokens_[i], distributionSpeeds_[i]);
}
```
This change will ensure that the length of the array is only accessed once, reducing the gas cost of the loop.
## References
```sol
2023-09-venus/contracts/Tokens/Prime/Prime.sol::176 => PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);
2023-09-venus/contracts/Tokens/Prime/Prime.sol::178 => for (uint256 i = 0; i < _allMarkets.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::204 => for (uint256 i = 0; i < users.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::211 => for (uint256 j = 0; j < _allMarkets.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::246 => for (uint256 i = 0; i < allMarkets.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::306 => _ensureMaxLoops(allMarkets.length);
2023-09-venus/contracts/Tokens/Prime/Prime.sol::335 => for (uint256 i = 0; i < users.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::349 => for (uint256 i = 0; i < users.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::609 => for (uint256 i = 0; i < _allMarkets.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::625 => for (uint256 i = 0; i < _allMarkets.length; ) {
2023-09-venus/contracts/Tokens/Prime/Prime.sol::730 => for (uint256 i = 0; i < _allMarkets.length; ) {
2023-09-venus/contracts/Tokens/Prime/PrimeLiquidityProvider.sol::88 => * @custom:error Throw InvalidArguments on different length of tokens and speeds array
2023-09-venus/contracts/Tokens/Prime/PrimeLiquidityProvider.sol::98 => uint256 numTokens = tokens_.length;
2023-09-venus/contracts/Tokens/Prime/PrimeLiquidityProvider.sol::99 => if (numTokens != distributionSpeeds_.length) {
2023-09-venus/contracts/Tokens/Prime/PrimeLiquidityProvider.sol::119 => for (uint256 i; i < tokens_.length; ) {
2023-09-venus/contracts/Tokens/Prime/PrimeLiquidityProvider.sol::151 => * @custom:error Throw InvalidArguments on different length of tokens and speeds array
2023-09-venus/contracts/Tokens/Prime/PrimeLiquidityProvider.sol::155 => uint256 numTokens = tokens_.length;
2023-09-venus/contracts/Tokens/Prime/PrimeLiquidityProvider.sol::157 => if (numTokens != distributionSpeeds_.length) {
2023-09-venus/contracts/Tokens/Prime/libs/FixedMath0x.sol::2 => // solhint-disable max-line-length
```