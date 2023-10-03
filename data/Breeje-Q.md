# QA Report

## Low Risk Issues
| Count | Explanation | 
|:--:|:-------|
| [L-01] | Solidity Version 0.8.13 used is vulnerable to optimizer issues |
| [L-02] | No functionality to change `maxLoopsLimit` can be problematic in long term |  
| [L-03] | `issue` doesn't check the 90 days staking condition to issue Prime Token |  
| [L-04] | Adding Multiple Markets with same underlying Tokens can be problematic |

| Total Low Risk Issues | 4 |
|:--:|:--:|

### [L-01] Solidity Version 0.8.13 used is vulnerable to optimizer issues

The solidity version 0.8.13 has below two issues:

1. Vulnerability related to ABI-encoding. ref : https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/
2. Vulnerability related to 'Optimizer Bug Regarding Memory Side Effects of Inline Assembly' ref : https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement/

Use recent Solidity version or atleast above 0.8.15 version which has the fix for these issues.

### [L-02] No functionality to change `maxLoopsLimit` can be problematic in long term

In `initialize` function of `Prime.sol` contract, `maxLoopsLimit` is set to `_loopsLimit` value through calling function `_setMaxLoopsLimit`.

```solidity
File: Prime.sol

`initialize` function:

164:    _setMaxLoopsLimit(_loopsLimit);

```

Issue here is that `_setMaxLoopsLimit` function is `internal` access function which cannot be called from outside and there is no other method in the contract which uses this function to change the value of `maxLoopsLimit` again in future to make sure no DoS issues happens.

Not having this access can lead to issues in future. So it is recommended to add a function that allows priviledged account to update this value.

### [L-03] `issue` doesn't check the 90 days staking condition to issue Prime Token

`issue` is a priviledged function used to Directly issue prime tokens to users.

But in implementation, the condition is not checked to make sure that user has staked the xvs tokens for 90 Days. This can lead to issuing of tokens even if user fails to satisfy the condition required to get the token.

```solidity
File: Prime.sol

  function issue(bool isIrrevocable, address[] calldata users) external {

    ***

            for (uint256 i = 0; i < users.length; ) {
                _mint(false, users[i]);
                _initializeMarkets(users[i]);
                delete stakedAt[users[i]]; // @audit no check for 90 Days staking condition.

                unchecked {
                    i++;
                }
     ***
    }

```

### [L-4] Adding Multiple Markets with same underlying Tokens can be problematic

`addMarket` function in `Prime` contract is used to Add a market to prime program.

Also there is a possibility that multiple markets have same underlying tokens.

If case it happens, the line below will overwrite any existing market.

```solidity
File: Prime.sol

    function addMarket(address vToken, uint256 supplyMultiplier, uint256 borrowMultiplier) external {

      ***

@->   vTokenForAsset[_getUnderlying(vToken)] = vToken;

      ***

    }

```

This can lead to issues as there will now be 2 markets in existance (`markets[vToken].exists = true`) with same underlying token but `vTokenForAsset` will only point to new market added.
