# QA Report

This contains all my low findings.

## PrimeLiquidityProvider sweepToken can transfer out accrued token rewards

### Code
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225

### Impact
The function `sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_)` can be used by the owner of PrimeLiquidityProvider to transfer out any amount of any ERC20 token.

It does not take into account the already accrued reward amounts for the token, which are stored in `tokenAmountAccrued[token_]`.

Only the current balance is checked against the provided amount, so the owner take any amount up to the contract's balance, including accrued reward amounts:
```
uint256 balance = token_.balanceOf(address(this));
if (amount_ > balance) {
    revert InsufficientBalance(amount_, balance);
}
```

Therefore the owner is able to take tokens that should be distributed to Prime token holders.

### Recommended Mitigation Steps
The function should check the request amount against the leftover balance (e.g. balance - accrued amount):
```
function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
    uint256 balanceAvailable = token_.balanceOf(address(this)) - tokenAmountAccrued[token_];
    if (amount_ > balanceAvailable) {
        revert InsufficientBalance(amount_, balanceAvailable);
    }

    emit SweepToken(address(token_), to_, amount_);

    token_.safeTransfer(to_, amount_);
}
```

## PrimeLiqduityProvider underflow reverts if balance is less than accrued rewards

### Code
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192-L205
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L261

### Impact
In PrimeLiquidityProvider there are some locations where total accrued rewards for a token is subtracted from the current balance. If the balance is lower than the rewards amount, an uncaught underflow revert will happen, blocking the functionality. This can happen with for example `sweepToken` or rebasing tokens.

For example, in `releaseFunds`:
```
uint256 accruedAmount = tokenAmountAccrued[token_];
tokenAmountAccrued[token_] = 0;
IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);
```
There is a transfer using the full amount, but there might be enough balance, causing `releaseFunds` to always revert until enough new tokens come in.

Also in `accrueRewards` on 261.

### Recommended Mitigation Steps
It would be safer to have `releaseFunds` try to send as much as possible and keep the leftover in `tokenAmountAccrued[token_]`, so that the Prime contract won't also revert when distributing interest. For example:
```
uint256 balance = token_.balanceOf(address(this));
uint256 accruedAmount = tokenAmountAccrued[token_];
uint256 leftoverAmount;
if (accruedAmount > balance) {
    leftoverAmount = accruedAmount - balance;
    accruedAmount = balance;
}
tokenAmountAccrued[token_] = leftoverAmount;
IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);
```

## Prime irrevocable tokens can still be burned

### Code
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L411-L414
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L725-L756

### Impact
The Prime tokens come in irrevocable and revocable versions. The update functions work correctly by not burning irrevocable tokens if the user is no longer eligible, but the `burn` function accepts tokens that are irrevocable and it can be called by anyone with the `burn` access control role.

### Recommended Mitigation Steps
If this is not intended, then the `_burn` function should check for the irrevocable flag and not allow burning of these tokens.

## Issueing of tokens to irrevocable Prime holders will revert

### Code
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L331-L359

### Impact
The `issue` function allows for the protocol to directly issue Prime tokens to users. If the user is already a revocable Prime holder, they will be upgraded to irrevocable. But if the user already holds an irrevocable Prime token, the function will incorrectly try to `mint` another token to the user on lines 337-342:
```
if (userToken.exists && !userToken.isIrrevocable) {
    _upgrade(users[i]);
} else {
    _mint(true, users[i]);
    _initializeMarkets(users[i]);
}
```
The conditional checks both existence and revocability, but it does not consider the case where it exists and is irrevocable, so it flows into the `else` branch, trying to `mint` another token.

The minting will revert, because users can only have a single token, causing the whole issuance for all users to be reverted.

The same applies to the other branch, for issuance of revocable tokens on lines 349-357.

### Recommended Mitigation Steps
The conditional branch should be changed such that the described case will just be ignored instead and not cause a revert. For example:
```
function issue(bool isIrrevocable, address[] calldata users) external {
    _checkAccessAllowed("issue(bool,address[])");

    if (isIrrevocable) {
        for (uint256 i = 0; i < users.length; ) {
            Token storage userToken = tokens[users[i]];
            if (userToken.exists) {
                if (!userToken.isIrrevocable) {
                    _upgrade(users[i]);
                }
            } else {
                _mint(true, users[i]);
                _initializeMarkets(users[i]);
            }

            unchecked {
                i++;
            }
        }
    } else {
        for (uint256 i = 0; i < users.length; ) {
            Token storage userToken = tokens[users[i]];
            if (!userToken.exists) {
                _mint(false, users[i]);
                _initializeMarkets(users[i]);
                delete stakedAt[users[i]];
            }

            unchecked {
                i++;
            }
        }
    }
}
```