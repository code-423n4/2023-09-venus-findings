### 1. USE SOLMATE INSTEAD OF OPENZEPPLIN
solmate is an efficient and optimized library of contracts maintained by transmissions, using solmate saved us more gas than using openzepplin contracts, you can find solmate contracts here: https://github.com/transmissions11/solmate/tree/main

INSTANCES IN CODE:
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L4
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L7

### 2. INSTEAD OF READING THE LENGTH OF AN ARRAY DIRECTLY FROM STORAGE WHEN USED IN A LOOP, WE CAN JUST CACHE IT AND FETCH IT DIRECTLY FROM THE STACK,THIS SAVES SO MUCH GAS SINCE TO REAM FROM THE STORAGE(WARM) WE USE 2100+ IN GAS AND WHEN READING FROM THE STACK WE USE 10+ IN GAS

EXAMPLES:
```
   function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
        address[] storage _allMarkets = allMarkets;
        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

        for (uint256 i = 0; i < _allMarkets.length; ) {
            address market = _allMarkets[i];
            uint256 interestAccrued = getInterestAccrued(market, user);
            uint256 accrued = interests[market][user].accrued;

            pendingInterests[i] = PendingInterest({
                market: IVToken(market).underlying(),
                amount: interestAccrued + accrued
            });

            unchecked {
                i++;
            }
        }
```
INSTEAD OF
```
   function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
        address[] storage _allMarkets = allMarkets;
        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);
        uint len =  _allMarkets.length
        for (uint256 i = 0; i < len; ) {
            address market = _allMarkets[i];
            uint256 interestAccrued = getInterestAccrued(market, user);
            uint256 accrued = interests[market][user].accrued;

            pendingInterests[i] = PendingInterest({
                market: IVToken(market).underlying(),
                amount: interestAccrued + accrued
            });

            unchecked {
                i++;
            }
        }
```

INSTANCES IN CODE:
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L178
 
### 3. USE PRE-INCREMENT INSTEAD OF POST-INCREMENT ``++i``` instead of ```i++```

```
contract test {
    function test21() public pure {//runtime cost is 193
    uint i;
       ++i;
    }
}

contract test1 {
    function red() public pure {//runtime cost is 198
        uint i;
        i++;
    }
}
```

INSTANCES IN CODE:
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L711
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L713C25-L713C25
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L766
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L819


### 4. USE PRE-DECREMENT INSTEAD OF POST-DECREMENT ``--i``` instead of ```i--```
EXAMPLES:
```
contract test {
    function test21() public pure {//runtime cost is 190 gas fees
    uint i = 7;
       --i;
    }
}

contract test1 {
    function red() public pure {//runtime cost is 195 gas fees
        uint i = 7;
        i--;
    }
}
```

INSTANCES IN CODE:
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L767
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L745
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L747
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L767
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L828
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L831
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L221


###5. DO NOT CACHE THE LENGTH OF ANY ARRAY STORED IN CALLDATA, IT'S  MORE EXPENSIVE AND COST THREE MORE GAS THAN USING IT DIRECT
```
contract test {
    function test21(uint256[] calldata _arr) public pure {// 1249 gas in runtime cost
         uint vars;
        for (uint256 i = 0; i < _arr.length; ++i) {
            vars = _arr[i];
        }
    }
}

contract test1 {
    function red(uint256[] calldata _arr) public pure {// 1254 gas in runtime cost
        uint  len = _arr.length;
        uint vars;
        for (uint256 i = 0; i < len; ++i) {
           vars = _arr[i];
        }
    }
}

```

INSTANCES IN CODE:
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L155
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L335
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L349

###6. THERE'S NO NEED TO CACHE VARIABLES IF THEY ARE ONLY USED ONCE

EXAMPLE:
```
contract test {
    uint public red;
    function test21(uint256[] calldata _arr) public view {//runtime gas cost: 2651
        if(red == _arr[0]){
            uint i;
            ++i;
        }
    }
}

contract test1 {
      uint public red;
    function test21(uint256[] calldata _arr) public view {//runtime gas cost: 2664
        uint cache = red;
        if(cache == _arr[0]){
            uint i;
            ++i;
        }
    }
}

```

INSTANCES IN CODE:
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L234
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L235
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L333

###7.CACHE VARIABLES YOU USE MORE THAN ONCE IN A FUNCTION ESPECIALLY IF THE VARIABLES ARE STATE VARIABLES(READ FROM STORAGE) OR ACCESSING A VALUE OF AN ARRAY FROM MEMORY AND YOU GET TO USE THIS VALUES MORE THAN ONCE THE BEST THING TO DO IS TO CACHE THEM

EXAMPLES:
```
contract test {
    uint public red;
    function test21(uint256[] calldata _arr) public view {//runtime cost is 4816 in gas cost
        if(red == _arr[0]){
            uint i = _arr[0] + red;
        }
    }
}

contract test1 {
      uint public red;
    function test21(uint256[] calldata _arr) public view {// 2729 in runtime gas cost
        uint cache = red;
        if(cache == _arr[0]){
            uint i = _arr[0] + cache;
        }
    }
}

```

INSTANCES IN CODE:
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L311
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L312
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L375
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L377
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L379
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L398
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L399
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L401
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L479
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L481
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L568
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L572
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L588

### 8. SINCE IT'S IMPOSSIBLE FOR THE STAKED AT TO BE GREATER THAN BLOCK.TIMESATMP, THEN WE CAN SIMPLY USE UNCHECKED FOR THE ARITHMETIC OPERATION

INSANCES IN CODE:
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L399C7-L399C7
