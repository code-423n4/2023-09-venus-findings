

# L-01 : Mistakenly sent underlying tokens in `PrimeLiquidityProvider.sol` may cause accounting issues in token accrual logic 

If an underlying token  is mistakenly sent to the `PrimeLiquidityProvider.sol`  , The `accrueTokens` function may miscalculate the token accrual because of using the contract balance for accounting .
```solidity 
 if (deltaBlocks > 0) {
            uint256 distributionSpeed = tokenDistributionSpeeds[token_];
            uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this)); //<----here 
```

 However if mistakenly sent tokens are sent back to the actual user through `sweepToken` function there's a possibility to temporary dos of the `accrueTokens` function cause it assumes the contract balance to be bigger than accrued amount of tokens . 



```solidity 
 if (deltaBlocks > 0) {
            uint256 distributionSpeed = tokenDistributionSpeeds[token_];
            uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this)); 

            uint256 balanceDiff = balance - tokenAmountAccrued[token_]; //<----here 
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L249

If the balance drops below the amount of accrued tokens , this will underflow and revert because of solidity 0.8.13 
This may result in temporary dos'ed state of `releaseFunds` function in `PrimeLiquidityProvider.sol` and `_claimInterest` function in `Prime.sol` 
# Recommendation 
Unsure about the mitigation . One way can be tracking  the valid txns to the  `PrimeLiquidityProvider.sol` in a variable and using that instead of ` balanceOf(address(this))` for accounting in `accrueTokens` function  . 


# L-02 :Solidity versions 0.8.13 and 0.8.14 are vulnerable to  reported optimizer bug related to inline assembly. 
The protocol uses solidity version  0.8.13 which is vulnerable to a reported  optimizer bug .
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L2 
# Recommendation 
Switching to a more stable version of solidity like 0.8.18 /  0.8.19 is recommended . 


# L-03 : Interest should only be claimed by the actual user  
As the current implementation of Prime anyone can call the `claimInterest` funcition an claim in behalf of an user .  Tokens are then sent to the actual user .  This may invoke certain scenarios where an user may forcefully receive the interest accrued by him even he didnot want to collect it 
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L443
# Recommendation 
Remove  `function claimInterest(address vToken, address user)` 

# L-04 : L2's like arbitrum's blocks are not consistant . Using `block.number' for token accrual accounting will result in different rewards for the same amount of token staked in different chains . 
L2's like arbitrum's blocks are not consistant . That's why Using `block.number' for token accrual accounting may result in different rewards for the same amount of token staked in different chains . 
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L254
```solidity 
 function accrueTokens(address token_) public {
        _ensureZeroAddress(token_);

        _ensureTokenInitialized(token_);

        uint256 blockNumber = getBlockNumber(); 
        uint256 deltaBlocks = blockNumber - lastAccruedBlock[token_]; //<---here 
```
# Recommendation 
Using block.timstamp is a better choice for consistant accounting in all chains . 

# NC-01 : Function naming is inaccurate 
`_ensureZeroAddress` function is to make sure the input address is not zero . But the naming is not correct according to its functionality . 
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L344

# Recommendation 
Naming should be  `_ensureNotZeroAddress` . 

