# Cache Array Length Outside of Loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Governance/GovernorAlpha.sol?plain=1#L219
```solidity
        for (uint i = 0; i < proposal.targets.length; i++) {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Governance/GovernorAlpha.sol?plain=1#L241

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Governance/GovernorAlpha.sol?plain=1#L265

## Recommended Mitigation Steps
```solidity
uint256 len = proposal.targets.length
for (uint i = 0; i < len; i++) {
    // invariant: array's length is not changed
}
```