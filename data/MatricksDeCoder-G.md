### GAS-1 Use variable packing 

Variables can be reduced in size e.g uint256 to smaller holding capacity in order to pack them in fewer slots especially in structs. See examples below 
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeStorage.sol#L7C5-L7C5
PrimeStorage.sol lines 5-26
```
    struct Market {
        uint256 supplyMultiplier;
        uint256 borrowMultiplier;
        uint256 rewardIndex;
        uint256 sumOfMembersScore;
        bool exists;
    }
// If the uint variable above are uint32 they are still large enough to contain the expected values. By making them uint32 we can pack them in a single slot 

    struct Interest {
        uint256 accrued;
        uint256 score;
        uint256 rewardIndex;
    }
// changing each above to uint64 can result in using a single storage slot instead of 3 

    struct PendingInterest {
        address market;
        uint256 amount;
    }

// making amount uint216 can still contain large amounts and allow packing with address which is 40 bits into a single slot of 256 
```

The above measures reduce the number of storage costs as SSTORE can cost up to 20,000 gas. However care is needed when slot packing to ensure its balanced with the cost fo operations to split the values when using them. The more the variables are used together in functions the more effective slot packing is justified. 

### GAS-2 Function names can be optimized 

Functions can be optimized for their signatures especially those beneficial for most used/most called functions in order to save gas. 

Optimizations have been seen to save in excess of over 100 gas per function for such optimizations 
Method IDs that have two leading zero bytes can save 128 gas each during deployment

There are various tools like below to optimize function names 
https://emn178.github.io/solidity-optimize-name/ 

### GAS-3 ++i costs less gas than i++ or --i cheaper than i--

There are several instances where more expensive post increment is used. These instances can be replaced with cheaper pre increment 

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L189
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L217
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L225
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L345
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L355
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L614
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L636
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L711
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L713
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L740
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L766
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L767
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L819

Preincrement ++i can be up to 5 times gas cheaper than i++ so the gas costs add up by looking at all the number cases the costs add up and should be saved. 

### GAS-3 Consider activating via-ir for deploying

IR-based code makes code generation more performant by enabling optimization passes that can be applied across functions. Include the option --via-ir or by including the option {"viaIR": true} or add --via-ir flag to your deploy command. 

### GAS-4 Remove initialization of i = 0 variable in for loops 

Several loops initialize the index variable i to default value 0 which wastes some memory gas in this reinitialization so an unnecessary 3 gas to update value to 0. The below are the instances 

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L178
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L204
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L211
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L246
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L335
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L349
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L609
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L625
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L730

Recommendation to change to the below 
```
drop the i=0 such that format is as below for all for loops 
uint length = arr.length;
for (uint i; i < length;) {
    unchecked { ++i; }
}
```

### GAS-5 Cache address(this) value 

There are instances in code where address(this) is used more than once in functions without caching the value. This results in unnecessary gas costs 

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L682C9-L690C10
Prime.so lines 682 - 690 
```
if (amount > asset.balanceOf(address(this))) {
            address[] memory assets = new address[](1);
            assets[0] = address(asset);
            IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
            if (amount > asset.balanceOf(address(this))) {
                IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
                unreleasedPLPIncome[underlying] = 0;
            }
        }
```

Its possible to go further and in constructor save address(this) as an immutable variable so that in all other instances its used even if not resued there is gas costs as immutable is like constants where values are added to bytecode; constants and immutable variables are cheaper. 

### GAS-6 Solidity Gas Optimizor set to default 200 runs 

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/hardhat.config.ts#L82
There is scope for gas savings by exploring the optimal optimizer runs settings for Solidity 

Not setting optimized value can result in using optimizer settings that are no best for reducing gas costs for the users. The more the runs e.g 10,000 the cheaper the gas costs for functions for users. The smaller the runs, the cheaper the deployment costs. It's essential to put users first in order to attract more and better usage of protocol.

### GAS-6 do while loops are cheaper than for loops

do while loops are cheaper than for loops; for loop will check the condition before executing the statement. A do while will execute the statement at least once before and then check the conditions, allowing for gas savings.

The following instances use for loops 
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L178
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L204
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L211
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L246
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L335
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L349
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L609
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L625
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L730
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L103
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L119
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L161







