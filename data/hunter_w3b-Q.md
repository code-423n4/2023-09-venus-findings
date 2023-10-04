## [L-01] Custom writting AccessControl

It is recommended to use OpenZeppelin AccessControl instead of custom-written AccessControlledV8.

```solidity
File: contracts/Tokens/Prime/Prime.sol


5   import { AccessControlledV8 } from "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L5

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol


5   import { AccessControlledV8 } from "@venusprotocol/governance-contracts/contracts/Governance/AccessControlledV8.sol";
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L5

## [L-02] Setter-functions must emit events

Emit events in setter functions

```solidity
File: Tokens/Prime/PrimeLiquidityProvider.sol

    function setTokensDistributionSpeed(address[] calldata tokens_, uint256[] calldata distributionSpeeds_) external {
        _checkAccessAllowed("setTokensDistributionSpeed(address[],uint256[])");
        uint256 numTokens = tokens_.length;

        if (numTokens != distributionSpeeds_.length) {
            revert InvalidArguments();
        }

        for (uint256 i; i < numTokens; ) {
            _ensureTokenInitialized(tokens_[i]);
            _setTokenDistributionSpeed(tokens_[i], distributionSpeeds_[i]);

            unchecked {
                ++i;
            }
        }
    }
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L153-L169

## [L-03] Dangerous usage of `block.timestamp` can be manipulated by miners.

The Prime.claim() function in the Prime contract relies on block.timestamp for two comparisons:

Checking if stakedAt[msg.sender] is equal to 0 (Line 398).
Checking if block.timestamp - stakedAt[msg.sender] is less than STAKING_PERIOD

This usage of block.timestamp for critical comparisons can be dangerous because block.timestamp can be manipulated by miners, potentially allowing malicious actors to exploit the function.

```solidity
File: contracts/Tokens/Prime/Prime.sol

    function claim() external {
        if (stakedAt[msg.sender] == 0) revert IneligibleToClaim();
        if (block.timestamp - stakedAt[msg.sender] < STAKING_PERIOD) revert WaitMoreTime();

        stakedAt[msg.sender] = 0;

        _mint(false, msg.sender);
        _initializeMarkets(msg.sender);
    }
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L397-L405

## [L-04] Strict Equality Check in Prime.claimTimeRemaining(address) Function

The Prime.claimTimeRemaining(address) function in the Prime contract uses a strict equality check to determine if stakedAt[user] is equal to 0 (Line 479). This strict equality check can be risky as it may be manipulated by an attacker.

```solidity
File:  contracts/Tokens/Prime/Prime.sol

479        if (stakedAt[user] == 0) return STAKING_PERIOD;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L479

it is advisable not to rely solely on strict equality checks to determine if an account has enough Ether or tokens. Instead, consider using other validation methods that are less prone to manipulation by attackers. This might involve implementing additional checks or conditions to ensure the correctness of the comparison.
