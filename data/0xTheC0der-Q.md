# Summary
* Low 1: Owner of `PrimeLiquidityProvider` contract can drain all tokens
* Low 2: `BLOCKS_PER_YEAR` is immuatble
* Low 3: `MAX_DISTRIBUTION_SPEED` is only intended for underlying assets with 18 decimals
* Low 4: Use of outdated OpenZeppelin library version
* Non-critical 1: Missing method `removeMarket(...)`
* Non-critical 2: No support for markets (`vToken`) with more than 18 decimals
* Non-critical 3: Hypothetical underflow


## Low 1: Owner of `PrimeLiquidityProvider` contract can drain all tokens
The method [PrimeLiquidityProvider.sweepToken(...)](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L207-L225) allows the contract owner to dain all its tokens.
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
Mitigation: Add a check to exclude underlying market tokens and therefore only allow stranded tokens to be withdrawn.

## Low 2: `BLOCKS_PER_YEAR` is immuatble
Having [Prime.BLOCKS_PER_YEAR](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L40) as an *immutable* value leads to the following concerns, since the **average amount of blocks per time** should be constant, but the reality looks like this: [Binance Smart Chain Blocks Per Day](https://ycharts.com/indicators/binance_smart_chain_blocks_per_day).
1. Although the amount of blocks per time should be constant, we cannot safely assume that this value is not subject to change during the course of a future upgrade. I recommend to add a setter function.
2. This value is an estimate at best and therefore only suitable for subsquent estimations, but not for calculations see [Prime._incomeDistributionYearly(...)](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L965-L979). I suggest to re-evaluate if `BLOCKS_PER_YEAR` is good enough for subsequent APR computations.

## Low 3: `MAX_DISTRIBUTION_SPEED` is only intended for underlying assets with 18 decimals
The [PrimeLiquidityProvider.MAX_DISTRIBUTION_SPEED](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L12) constant is `1e18` and therefore implicitly allows faster assets distributions for assets having less than 18 decimals, see also [PrimeLiquidityProvider._setTokenDistributionSpeed(...)](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L303-L326).

## Low 4: Use of outdated OpenZeppelin library version
According to [package.json](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/package.json#L34-L35), `openzeppelin-contracts v4.8.3` and `openzeppelin-contracts-upgradeable v4.8.0` is used. Both versions are outdated and have known vulnerabilities, see [Snyk Vulnerability Database](https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts).

## Non-critical 1: Missing method `removeMarket(...)`
There is a method [Prime.addMarket(...)](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L282-L309). However, it might be required in some occasions to remove a market (`vToken`) from `Prime`.

## Non-critical 2: No support for markets (`vToken`) with more than 18 decimals
The `vToken` [initializer](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/VTokens/VToken.sol#L305-L347) allows to create markets with user-defined decimals. However, [L661](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L661) in [Prime._calculateScore(...)](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L661) allows the usage of markets with less than or equal to 18 decimals only. Underflow error in case of more than 18 decimals.
```solidity
    function _calculateScore(address market, address user) internal returns (uint256) {
        ...
        capital = capital * (10 ** (18 - vToken.decimals()));
        ...
    }
```

## Non-critical 3: Hypothetical underflow
In line [L800](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L800) of [Prime._updateScore(...)](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L789-L802)
```solidity
    function _updateScore(address user, address market) internal {
        ...
        markets[market].sumOfMembersScore = markets[market].sumOfMembersScore - interests[market][user].score + score;
        ...
    }
```
there might happen an underflow leading to DoS in case `interests[market][user].score > markets[market].sumOfMembersScore`. Although such a case seems not possible due to the current contract logic, I still recommend to rewrite as:
```solidity
    function _updateScore(address user, address market) internal {
        ...
        markets[market].sumOfMembersScore = (markets[market].sumOfMembersScore + score) - interests[market][user].score;
        ...
    }
```