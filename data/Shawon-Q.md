* L682 of `_claimInterest` function,there is a if statement `if (amount > asset.balanceOf(address(this)))` .Then again,inside the mentioned 
if statement there is again the same check on L686 which is unnecessary as this part of code will only execute if  the same check  mentioned above.

* 
There are several uint256 variables inside PrimeStorage.sol which can be set as immutable.such as,
 ``` solidity 
    /// @notice  Tracks total revocable tokens minted
    uint256 public totalRevocable;

    /// @notice  Indicates maximum revocable tokens that can be minted
    uint256 public revocableLimit;

    /// @notice  Indicates maximum irrevocable tokens that can be minted
    uint256 public irrevocableLimit;
```