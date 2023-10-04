
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| O | Ordinary | Often found issues |

| Total Found Issues | 4 |
|:--:|:--:|


### Low Risk 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-00] |Either Revocable / Irrevocable tokens cannot be minted if  one reaches the maxLimit | 1 |
| [L-00] | In `Pime.sol` `xvsUpdated()` function could be called by any user| 1 |
| [L-01] | Prime Token can be issued to zero address    | 1 |


### Non-Critical 

| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-00] |In `Prime.sol::_burn()` it's  not essential to set isIrrevocable false if it is already false| 1 |


## L-00  Either Revocable / Irrevocable tokens cannot be minted if  one reaches the maxLimit 

## Summary 

If one  type of Prime Token reach the limit we can't mint the other type 


## Vulnerability in  details 


The limit for minting  revocable and irrevocable  Prime tokens is ,
 
 ```js
    /// @notice  Indicates maximum revocable tokens that can be minted
    uint256 public revocableLimit;

    /// @notice  Indicates maximum irrevocable tokens that can be minted
    uint256 public irrevocableLimit;


```
It is checked in `_mint` function as below [Prime.sol lineL716](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L716)  ,and lets  say 
```
 revocableLimit  = 10   
 irrevocableLimit =  10 
```       


```js
...
    if (isIrrevocable) {
            totalIrrevocable++;
        } else {
            totalRevocable++;
        }

        if (totalIrrevocable > irrevocableLimit || totalRevocable > revocableLimit) revert InvalidLimit();
//                  11  > 10                            5 > 10 
//                    true                      ||        false     => true   therefore revert 

        emit Mint(user, isIrrevocable);

```
So that means if either revocable / Irrevocable reach the limit we can't mint the other type   


### Tools Used 

Manual Review 

### Mitigation Steps 
Consider ,

```diff
...
    if (isIrrevocable) {
            totalIrrevocable++;
+            if (totalIrrevocable > irrevocableLimit ) revert InvalidLimit();
        } else {
            totalRevocable++;
+             if(totalRevocable > revocableLimit) revert InvalidLimit();
        }

-        if (totalIrrevocable > irrevocableLimit || totalRevocable > revocableLimit) revert InvalidLimit();
       

        emit Mint(user, isIrrevocable);

```


## L-01   In `Pime.sol` `xvsUpdated()` function can be called by arbitrary user

It is said that, 
 ``` 
 /**
     * @notice Executed by XVSVault whenever user's XVSVault balance changes
     * @param user the account address whose balance was updated
     */
```
However anyone can call  the function  `xvsUpdated()` to burn revocable ineligible users or update the interest score of eligible users  as because it  doesn't check that the caller is only the  `XVSVault`

Consider ,

```diff
function xvsUpdated(address user) external {
+  if(msg.sender != xvsVault) {
+    revert NotXVSVault();
+ }    
        uint256 totalStaked = _xvsBalanceOfUser(user);

or add access allowed check just like in burn function 

 function burn(address user) external {
        _checkAccessAllowed("burn(address)");
        _burn(user);
    }

```

## L-02  Prime Token can be minted to zero address  

 A user can have only a one primeToken so in the `_mint`  function  it checks that the token  already minted or not by 
  ```js
  function _mint(bool isIrrevocable, address user) internal {
        if (tokens[user].exists) revert IneligibleToClaim();
  ```      
however it doesn't check that user is zero address, 

## Tool used
Manual Review

## Recommendation

consider ,

```diff
unction _mint(bool isIrrevocable, address user) internal {
+           if(user == address(0)) revert Mint_to_Zero_Address();  
        if (tokens[user].exists) revert IneligibleToClaim();

```

## N-00 In `Prime.sol::_burn()` it's  not essential to set isIrrevocable false if it is already false

```diff

  if (tokens[user].isIrrevocable) {
            totalIrrevocable--;
+          tokens[user].isIrrevocable = false;    
        } else {
            totalRevocable--;
        }

        tokens[user].exists = false;
-        tokens[user].isIrrevocable = false;

        _updateRoundAfterTokenBurned(user);

        emit Burn(user);

```

