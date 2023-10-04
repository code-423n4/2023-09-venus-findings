### [Low Risk]

| Count | Title |
| --- | --- |
| [L-01] | updateScores will result in DoS if pass a user with an already updated score |
| [L-02] | sweepToken cannot be called for blacklisted to_ users |

| Total Low Risk Issues | 2 |
| --- | --- |

### [Non-Critical]

| Count | Title |
| --- | --- |
| [NC-01] | Use isIrrevocable instead of true/false |
| [NC-02] | PrimeTokenUpdated event should be emitted after update the state variable |

| Total Non-Critical Issues | 2 |
| --- | --- |

## Low Risk

## [L-01] `updateScores` will result in DoS if pass a user with **an already updated score**

## Impact

If `updateScores` is called for a user who is already updated in the same round, the function will misbehave, causing it to repeat lines 205-208 until the gas limit is reached.

```solidity
200:	function updateScores(address[] memory users) external {
        if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
        if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

        for (uint256 i = 0; i < users.length; ) {
205:        address user = users[i];
206:	
207:        if (!tokens[user].exists) revert UserHasNoPrimeToken();
            //@audit in case user who has his score updated is passed to the array `i` will not be incremented which will lead to the waste of gas when function is called 
208:        if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

            address[] storage _allMarkets = allMarkets;
            for (uint256 j = 0; j < _allMarkets.length; ) {
                address market = _allMarkets[j];
                _executeBoost(user, market);
                _updateScore(user, market);

                unchecked {
                    j++;
                }
            }

            pendingScoreUpdates--;
            isScoreUpdated[nextScoreUpdateRoundId][user] = true;

            unchecked {
                i++;
            }

            emit UserScoreUpdated(user);
        }
    }
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L208

## [L-02] `sweepToken` cannot be called for blacklisted `to_` users

## Impact

If the admin attempts to sweep tokens to a user blacklisted for the specific token, the tokens will remain locked in the contract until the user provides an alternative receiving address.

```solidity
function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
    uint256 balance = token_.balanceOf(address(this));
    if (amount_ > balance) {
        revert InsufficientBalance(amount_, balance);
    }

    emit SweepToken(address(token_), to_, amount_);

    token_.safeTransfer(to_, amount_);
}
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225

## Non-Critical

## **[N‑01] Use isIrrevocable instead of true/false**

Use **isIrrevocable** parameter instead of boolean literals.

```solidity
File: contracts/Tokens/Prime/Prime.sol

331: function issue(bool isIrrevocable, address[] calldata users) external {
    _checkAccessAllowed("issue(bool,address[])");

    if (isIrrevocable) {
        for (uint256 i = 0; i < users.length; ) {
            Token storage userToken = tokens[users[i]];
            if (userToken.exists && !userToken.isIrrevocable) {
                _upgrade(users[i]);
            } else {
                // @audit use isIrrevocable
                _mint(true, users[i]);
                _initializeMarkets(users[i]);
            }

            unchecked {
                i++;
            }
        }
    } else {
        for (uint256 i = 0; i < users.length; ) {
            // @audit use isIrrevocable
            _mint(false, users[i]);
            _initializeMarkets(users[i]);
            delete stakedAt[users[i]];

            unchecked {
                i++;
            }
        }
    }
}
```

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L340

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L350

## [N-02] PrimeTokenUpdated event should be emitted after update the state variable

PrimeTokenUpdated event should be emitted at the end of the function `PrimeLiquidityProvider.sol:setPrimeToken()`.

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177-L182