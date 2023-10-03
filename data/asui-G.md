1. In the ```updateAssetsState``` function in Prime.sol contract instead of doing this :
   ```
   IVToken market = IVToken(vToken); 
   unreleasedPSRIncome[_getUnderlying(address(market))] = 0;
   ``` 
   directly call this: ```unreleasedPSRIncome[_getUnderlying(vToken)] = 0;``` since ``` IVToken market = 
   IVToken(vToken);``` is never really used in that function . The function is unnecessarily storing the 
   ```vToken``` to an ```IVToken``` interface type variable called ```market``` which is never used but instead it 
   is turned back to an address type ```address(market)``` converting to and fro unnecessarily. 