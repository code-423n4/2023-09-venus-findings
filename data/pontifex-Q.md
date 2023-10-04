### L-1 Wrong comment about function accessability
The `sweepToken` function of the `PrimeLiquidityProvider` contract has the `onlyOwner` modifier, but in comments the function is described as a public function.
```solidity
208     * @notice A public function to sweep accidental ERC-20 transfers to this contract. Tokens are sent to user


216    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L208


### L-2 Consider function optimization
Consider the `xvsUpdated` function optimization to improve both readability and gas economy.
The current implementation:
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L365-L382
```solidity
    function xvsUpdated(address user) external {
        uint256 totalStaked = _xvsBalanceOfUser(user);
        bool isAccountEligible = isEligible(totalStaked);


        if (tokens[user].exists && !isAccountEligible) {
            if (tokens[user].isIrrevocable) {
                _accrueInterestAndUpdateScore(user);
            } else {
                _burn(user);
            }
        } else if (!isAccountEligible && !tokens[user].exists && stakedAt[user] > 0) {
            stakedAt[user] = 0;
        } else if (stakedAt[user] == 0 && isAccountEligible && !tokens[user].exists) {
            stakedAt[user] = block.timestamp;
        } else if (tokens[user].exists && isAccountEligible) {
            _accrueInterestAndUpdateScore(user);
        }
    }
```
The proposed implementation:
```solidity
    function xvsUpdated(address user) external {
        uint256 totalStaked = _xvsBalanceOfUser(user);
        bool isAccountEligible = isEligible(totalStaked);


        if (tokens[user].exists) {
            if (isAccountEligible || tokens[user].isIrrevocable) {
                _accrueInterestAndUpdateScore(user);
            } else {
               _burn(user);
            }
        } else if (stakedAt[user] == 0 && isAccountEligible) {
            stakedAt[user] = block.timestamp;
        } else if (!isAccountEligible && stakedAt[user] > 0) {
            stakedAt[user] = 0;
        }
    }
```

### N-1 Typos
There are 4 instances:
The word `interes` should be `interest`.
```solidity
385     * @notice accrues interes and updates score for an user for a specific market


604     * @notice accrues interes and updates score of all markets for an user
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L385


The word `intialized` should be `initialized`.
```solidity
115     * @param tokens_ Array of addresses of the tokens to be intialized


282     * @param token_ Address of the token to be intialized
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L115




### N-2 `constant` should be defined rather than using magic number
Use a readable constant instead of a numeric value.
```solidity
661        capital = capital * (10 ** (18 - vToken.decimals()));
```
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L661