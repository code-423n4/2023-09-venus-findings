## QA REPORT

| |Issue|
|-|:-|
| [01] | `getEffectiveDistributionSpeed()` function of PrimeLiquidityProvider should check for balance == 0 |
| [02] | Use i++ consistently |
| [03] | Underscore can be added for large numbers in PrimeStorage.sol |
| [04] | Use "days" instead of magic numbers |
| [05] | Use whenNotPaused modifer instead of if(paused()) |
| [06] | _calculateScore() of Prime.sol should consider larger decimals then 18 |
| [07] | Prime.sol does not have any way of removing Markets |

## [01] `getEffectiveDistributionSpeed()` function of PrimeLiquidityProvider should check for balance == 0

Inside of the getEffectiveDistributionSpeed() inside of PrimeLiquidityProvider a missing check for zero balance will lead the function to revert in certain cases. This lead to several reverts in Prime.sol, bricking multiple view functions.
This does not lead to loss of funds, but can be anoying for the user. For this reason the report is included as a "low" report inside of this QA. It can be argued that this is a medium severity.

```solidity
function getEffectiveDistributionSpeed(address token_) external view returns (uint256) {
        uint256 distributionSpeed = tokenDistributionSpeeds[token_];
        uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
        uint256 accrued = tokenAmountAccrued[token_];

        if (balance - accrued > 0) {
            return distributionSpeed;
        }

        return 0;
    }
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L232-L242

as we can see the check balance - accrued will revert if balance is smaller then accrued. This can be easily the case because balance gets completely withdrawn if the accureToken function withdraws the remaining balance:
To note: the accrueTokens() function can be called indipendantly from the releaseFunds function. This wont reset the tokenAmountAccrued mapping.

```solidity
...
            uint256 balanceDiff = balance - tokenAmountAccrued[token_];
            if (distributionSpeed > 0 && balanceDiff > 0) {
                uint256 accruedSinceUpdate = deltaBlocks * distributionSpeed;
                uint256 tokenAccrued = (balanceDiff <= accruedSinceUpdate ? balanceDiff : accruedSinceUpdate);
...
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L261-L264

Following view functions would revert because of this:
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L970
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L993
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527

## [02] Use i++ consistently

PrimeLiquidityProvider.sol uses ++i and Prime.sol uses i++. Its a common practice to keep codebase consistent.

```solidity
unchecked {
            ++i;
          }
```
Following lines are affected:
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L107-L109
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L122-L124
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L165-L167

## [03] Underscore can be added in PrimeStorage.sol 
It is a common practice to separate each 3 digits in a number by an underscore to improve code readability. 100000 can be updated to 100_000 and 10000 to 10_000 inside of PrimeStorage.sol.

```solidity
    uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;
    ....
    uint256 internal constant MAXIMUM_BPS = 10000;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L37
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L43

## [04] Use "days" instead of magic numbers
Inside of PrimeStorage the STAKING_PERIOD constant can be changed to "90 days" instead of calculation.
```solidity
    /// @notice number of days user need to stake to claim prime token
    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L40

## [05] Use whenNotPaused modifer instead of if(paused())
Inside of Prime/PrimeLiquidityProvider the releaseFunds() function should use the whenNotPaused modifer instead of a custom if().

```solidity
        function releaseFunds(address token_) external {
        if (msg.sender != prime) revert InvalidCaller();
        if (paused()) {
            revert FundsTransferIsPaused();
        }
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192

## [06] _calculateScore() of Prime.sol should consider larger decimals then 18
The _calculateScore function of Prime.sol uses the .decimals() of the current vToken to calculate capital. 
This calculation will underflow if the decimals returned are larger then 18.
The case where is applies to in real world will be seen, but when considering smaller decimals then 18 the oposit case of larger decimals should be considered as well.

```solidity
                (uint256 capital, , ) = _capitalForScore(xvsBalanceForScore, borrow, supply, market);
				capital = capital * (10 ** (18 - vToken.decimals())); // will revert if more then 18 decimals
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L661

If there is ever a market with a token with more decimals added, the impact will be high because no user can mint prime tokens anymore.
Because there is no function to remove markets, this will brick the contract for ever.

## [07] Prime.sol does not have any way of removing markets
If there is any issue in one of Primetokens markets, these can include security issues, 
upgrade issues or as mentioned before other decimals then < 18 it will lead to several problems inside of Prime.sol.
Most impactfull total revert of xvsUpdated, bricking the contract.

Currently there is only a way to add markets, but not way to remove them.
Assuming venus will only use markets that have no issues and have 18 or less decimals, this being a problem is quit unlikly. For this reason the issue is considered a "Low".

The recomendation is still to include some way of removing/disabling certained markets, to make the contract less depended on external state.

