| *Issue* | *Description*                                                                                                                                                                                                                                                                    |
|---------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [L-01]  | [estimateAPR](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548) can calculate slightly wrong APR                                                                                                                                     |
| [L-02]  | [sweepToken](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225) can break [releaseFunds](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192-L205) and [accrueTokens](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249-L272)|
| [L-03]  | [calculateAPR](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496-L515) will return overinflated values |
| [L-04]  | There is no current way to remove boosted markets |
| [L-05]  | [getPendingInterests](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L194) and [_accrueInterestAndUpdateScore](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L607-L617) can run out of gas |
| [L-06]  | [issue](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359) does not delete [stakedAt](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L352) mapping                                        |
| [N-01]  | Admins can [burn](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L411-L414) users Irrevocable tokens                                                                                                                                     |


### [L-01] [estimateAPR](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548) can calculate slightly wrong APR
 If [estimateAPR](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548) is used by user who already has staked his balance into the system, [estimateAPR](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548) will return a wrong and slightly lower **APR**. This is due to the system calculating a new user score and adding it to the already existing user scores. In `totalScore` if the user has staked, his score is already accounted for, and when [estimateAPR](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548) calculates his new score and adds it in, our user's score becomes counted twice. Thus inflating `totalScore` value slightly which results in lower APR. 


### [L-02] [sweepToken](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225) can break [releaseFunds](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192-L205) and [accrueTokens](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249-L272)

If [sweepToken](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225) is called on a token that is used in `tokenAmountAccrued[token_]` it's balance will get lower than `tokenAmountAccrued[token_]`. This in tern will break every accounting that uses `tokenAmountAccrued[token_]`. 

Example :

1. [accrueTokens](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249-L272) will underflow as balance will be 0 and `tokenAmountAccrued[token_]` will be some amount.

```jsx
            uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
            uint256 balanceDiff = balance - tokenAmountAccrued[token_];
```

2. [releaseFunds](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192-L205) will revert due to trying to send more tokens that it has.

```jsx
        uint256 accruedAmount = tokenAmountAccrued[token_];
        tokenAmountAccrued[token_] = 0;
        IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);
```

If [sweepToken](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225) is called on a currently used reward token it will break it's functionality and there will be no way to fix it. This is why it is best to have some protection against such things.

```diff    
function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
        uint256 balance = token_.balanceOf(address(this));
        if (amount_ > balance) {
            revert InsufficientBalance(amount_, balance);
        }

+       if(tokenAmountAccrued[token_] > 0){
+           revert TokenInUse(token_);
+       }

        emit SweepToken(address(token_), to_, amount_);
        token_.safeTransfer(to_, amount_);
    }
```

### [L-03][calculateAPR](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496-L515) will return overinflated values
[calculateAPR](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496-L515) and [estimateAPR](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548) will return overinflated values due to wrong returns of PLP.
Flow: [calculateAPR](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496-L515) => [_calculateUserAPR](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L993-L1014) => [_incomeDistributionYearly](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L970-L979). [**_incomeDistributionYearly**](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L970-L979) combines the earning of PSR and PLP, however the earnings of PLP will be wrong as when [getEffectiveDistributionSpeed](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L976-L977) is called it will return `tokenDistributionSpeeds[token_]` which is the fixed value of the income. While [accrueTokens](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249-L272) compares which values are lower:

 - IERC20Upgradeable(token_).balanceOf(address(this))
 - tokenDistributionSpeeds[token_]



And assigns the lower one. 

```jsx
            uint256 balanceDiff = balance - tokenAmountAccrued[token_];
            if (distributionSpeed > 0 && balanceDiff > 0) {
                uint256 accruedSinceUpdate = deltaBlocks * distributionSpeed;
                uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);
                tokenAmountAccrued[token_] += tokenAccrued;
```

This means that if sometimes `balance` is lower than `tokenDistributionSpeeds` the APR will be inflated, and with `balance` being higher the APR to be underinflated.

### [L-04] There is no current way to remove boosted markets
Markets can only be added to the system without a way to remove them. For now this is not an issue as only well used and big markets will be added (BTC,ETH,USDT,USDC <- mentioned by the dev.), however in the future if smaller markets are included and they die off, it will be beneficial for them to be removed. To save gas on the loops with `_allMarkets.length` and for easier management. 

### [L-05] [getPendingInterests](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L194) and [_accrueInterestAndUpdateScore](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L607-L617) can run out of gas
As all places where `_allMarkets.length` is used can be DoS partly as the amount of markets can increase in the future. I would suggest to store and use only markets that a user participates in. This can be easily achieved by a single mapping where it is stored `user => market`.

### [L-06] [issue](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359) does not delete [stakedAt](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L352) mapping
 This inconsistency in issue](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359 can cause some issues in the future or mess with the time mapping where the staking duration is stored. As one of the paths under issue](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359 does not delete `stakedAt[]`, like the other paths.

```diff
    function issue(bool isIrrevocable, address[] calldata users) external {
        _checkAccessAllowed("issue(bool,address[])");

        if (isIrrevocable) {
            for (uint256 i = 0; i < users.length; ) {
                Token storage userToken = tokens[users[i]];
                if (userToken.exists && !userToken.isIrrevocable) {
                    _upgrade(users[i]);
                } else {
                    _mint(true, users[i]);
                    _initializeMarkets(users[i]);
+                   delete stakedAt[users[i]];//@audit L `stakedAt` not deleted
                }

                unchecked {
                    i++;
                }
            }
        } else {
            for (uint256 i = 0; i < users.length; ) {
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

### [N-01] Admins can [burn](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L411-L414) users Irrevocable tokens
In the code, and explained in discord it was stated that Irrevocable are not destroyable and not burnable, however the admins of the system are able to burn them. Of course this is not a major issue since the admins are not malicious, but it's gonna be useful to state it in the docs that if users misbehave admins are able to burn these tokens. 