* https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L111 refers to the Prime contract being upgradable. However, it is not upgradable.

* At https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L184 , it uses `.underlying` to ge the underlying asset of a vToken. However, this does not work for VBNB. Use `Prime._getUnderlying` instead.

* The code at https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L661 does not work if a vToken has more than 18 decimals.