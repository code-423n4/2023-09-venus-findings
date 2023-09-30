### [L-01] Weak Sources of Randomness from Chain Attributes
## Impact 
**Potential use of "block.number" as source of randomness.**
The environment variable "block.number" looks like it might be used as a source of randomness. 
Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. 
Also keep in mind that attackers know hashes of earlier blocks. 
**Recommendation**
Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
Using external sources of randomness via oracles, and cryptographically checking the outcome of the oracle on-chain. e.g. Chainlink VRF. This approach does not rely on trusting the oracle, as a falsly generated random number will be rejected by the on-chain portion of the system.
Using commitment scheme, e.g. RANDAO.
Using external sources of randomness via oracles, e.g. Oraclize. Note that this approach requires trusting in oracle, thus it may be reasonable to use multiple oracles.
Using Bitcoin block hashes, as they are more expensive to mine.
## Vulnerable Function
```sol
// https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L276-L278
    function getBlockNumber() public view virtual returns (uint256) {
        return block.number;
    }
```
## Vulnerable Code Snippet
```sol
// https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L277
        return block.number;
```