| *Issue* | *Description*                                                                                                                                                                                                                                                                    |
|---------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [L-01]  | There is no current way to remove boosted markets                                                                                                                                                                                                                                |
| [L-02]  | [getPendingInterests](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L194) and [_accrueInterestAndUpdateScore](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L607-L617) can run out of gas |
| [L-03]  | [issue](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359) does not delete [stakedAt](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L352) mapping                                        |
| [L-04]  | Admins can [burn](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L411-L414) users Irrevocable tokens                                                                                                                                     |

### [L-01] There is no current way to remove boosted markets
Markets can only be added to the system without a way to remove them. For now this is not an issue as only well used and big markets will be added (BTC,ETH,USDT,USDC <- mentioned by the dev.), however in the future if smaller markets are included and they die off, it will be beneficial for them to be removed. To save gas on the loops with `_allMarkets.length` and for easier management. 

### [L-02] [getPendingInterests](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L194) and [_accrueInterestAndUpdateScore](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L607-L617) can run out of gas
As all places where `_allMarkets.length` is used can be DoS partly as the amount of markets can increase in the future. I would suggest to store and use only markets that a user participates in. This can be easily achieved by a single mapping where it is stored `user => market`.

### [L-03] [issue](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331-L359) does not delete [stakedAt](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L352) mapping
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

### [L-04] Admins can [burn](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L411-L414) users Irrevocable tokens
In the code, and explained in discord it was stated that Irrevocable are not destroyable and not burnable, however the admins of the system are able to burn them. Of course this is not a major issue since the admins are not malicious, but it's gonna be useful to state it in the docs that if users misbehave admins are able to burn these tokens. 