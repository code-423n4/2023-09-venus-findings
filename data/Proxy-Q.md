### [Non-Critical](#non-critical-1)

| Total Non-Critical Issues | 3 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [NC-01](#nc-01-use-a-more-descriptive-name-for-claiming-prime-token) | Use a more descriptive name for claiming prime token | 1 |
| [NC-02](#nc-02-wrong-description) | Wrong description | 1 |
| [NC-03](#nc-03-misleading-function-name) | Misleading function name | 1 |

## Non-critical

### [NC-01] Use a more descriptive name for claiming prime token

Prime.sol contract uses a lot of functions starting with claim. And just using `claim()` is not very descriptive and can be misleading.

[Prime.sol#L397](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L397)

Change to: `claimPrimeToken()` or `claimToken()`

### [NC-02] Wrong description

State variable `lastAccruedBlock` uses the wrong description.

[PrimeLiquidityProvider#L23](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L23)

```solidity
/// @notice The rate at which token is distributed to the Prime contract
mapping(address => uint256) public lastAccruedBlock;
```

Change description to: The last block number where the token interest has been accrued

### [NC-03] Misleading function name

Function `_ensureZeroAddress()` reverts if address provided is zero, however the name suggests that the address must be zero address. [Ensure meaning](https://www.google.com/search?q=ensure+meaning&sca_esv=570303733&sxsrf=AM9HkKniRRDkXnfdC3EVCN2c1Xj0d4Qipw%3A1696330048103&ei=QPEbZfjqBYmCxc8PmKG3gAw&ved=0ahUKEwj4isHv2dmBAxUJQfEDHZjQDcAQ4dUDCBA&uact=5&oq=ensure+meaning&gs_lp=Egxnd3Mtd2l6LXNlcnAiDmVuc3VyZSBtZWFuaW5nMgcQIxiKBRgnMggQABjLARiABDIIEAAYywEYgAQyCBAAGMsBGIAEMggQABjLARiABDIIEAAYywEYgAQyCBAAGMsBGIAEMggQABjLARiABDIIEAAYywEYgAQyCBAAGMsBGIAESPIKUOMCWPYJcAF4AZABAJgBgAGgAf8GqgEDMC44uAEDyAEA-AEBwgIKEAAYRxjWBBiwA8ICChAAGIoFGLADGEPCAgUQABiABMICBxAAGIoFGEPiAwQYACBBiAYBkAYK&sclient=gws-wiz-serp).

Change name to `_ensureNotZeroAddress()`.