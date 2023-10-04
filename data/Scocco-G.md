# Cache Array Length to Save Gas

## Impact

Reading the length of the array in each iteration of the loop takes 6 gases (3 for mload and 3 for putting memory_offset) on the stack. Caching the array length on the stack saves about 3 gas per iteration.

## Proof of Concept

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L178
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L204
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L211
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L246
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L335
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L349
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L609
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L625
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L730

Lines:
```solidity
178 ┆ for (uint256 i = 0; i < _allMarkets.length; ) {
  ⋮┆----------------------------------------
204 ┆ for (uint256 i = 0; i < users.length; ) {
  ⋮┆----------------------------------------
211 ┆ for (uint256 j = 0; j < _allMarkets.length; ) {
  ⋮┆----------------------------------------
246 ┆ for (uint256 i = 0; i < allMarkets.length; ) {
  ⋮┆----------------------------------------
335 ┆ for (uint256 i = 0; i < users.length; ) {
  ⋮┆----------------------------------------
349 ┆ for (uint256 i = 0; i < users.length; ) {
  ⋮┆----------------------------------------
609 ┆ for (uint256 i = 0; i < _allMarkets.length; ) {
  ⋮┆----------------------------------------
625 ┆ for (uint256 i = 0; i < _allMarkets.length; ) {
  ⋮┆----------------------------------------
730 ┆ for (uint256 i = 0; i < _allMarkets.length; ) {

```
## Tools Used

Manual code analysis

## Recommended Mitigation steps

I recommend a change from:
```solidity
for (uint256 i=0; i<array.length; i++) { ... }
```

To:
```solidity
uint len = array.length  
for (uint256 i=0; i<len; i++) { ... }
```