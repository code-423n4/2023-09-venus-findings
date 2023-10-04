# Gas Optimization

While striving for enhanced code clarity in the provided snippets, certain functions have been abbreviated to emphasize affected sections.

Developers should remain vigilant during the incorporation of these proposed modifications to avert potential vulnerabilities. Despite prior testing of the optimizations, developers bear the responsibility of conducting comprehensive reevaluation.

Conducting code reviews and additional testing is highly recommended to mitigate any plausible hazards that may arise from the refactoring endeavor.

# Summary

| Number | Issues                                                                      | Instances |
| :----: | :-------------------------------------------------------------------------- | :-------: |
| [G-01] | Use ++i instead of i++ to increment                                         |    19     |
| [G-02] | Admin functions can be payable                                              |     3     |
| [G-03] | Always use Named Returns                                                    |    20     |
| [G-04] | Make the variable outside the loop to save gas                              |     6     |
| [G-05] | Revert() statements should be used sorted from cheapest to most expensive   |     2     |
| [G-06] | Should use arguments instead of the state variables                         |     3     |
| [G-07] | Amounts should be checked for `0` before calling a transfer                 |     1     |
| [G-08] | Use bitmaps instead of bools when a significant amount of booleans are used |     8     |
| [G-09] | Using Storage instead of memory for structs/arrays saves gas                |     2     |
| [G-10] | Do not calculate constants                                                  |     3     |
| [G-11] | block.number directly call instead of a function                            |     1     |
| [G-12] | Use do while loops instead of for loops                                     |    12     |
| [G-13] | Use assembly to perform efficient back-to-back calls                        |    17     |
| [G-14] | Ternary operation is cheaper than if-else statement                         |     4     |
| [G-15] | Short-circuit booleans                                                      |    10     |
| [G-16] | Unnecessary casting as variable is already of the same type                 |     2     |
| [G-17] | Use hardcode address instead address(this)                                  |     7     |
| [G-18] | Using mappings instead of arrays to avoid length checks                     |     6     |
| [G-19] | Using delete statement can save gas                                         |    10     |
| [G-20] | Shorten the array rather than copying to a new one                          |     2     |
| [G-21] | Use v4.9.0 OpenZeppelin contracts                                           |     1     |
| [G-22] | Mappings used within a function more than once should be cached to save gas |    28     |
| [G-23] | Divisions can be unchecked to save gas from previous check                  |     3     |
| [G-24] | Initializers can be marked as payable to save deployment gas                |     2     |
| [G-25] | Consider using alternatives to OpenZeppelin                                 |     6     |
| [G-26] | Using assembly to revert with an error message                              |    39     |
| [G-27] | Split revert statements                                                     |     3     |
| [G-28] | Make event parameters indexed when possible                                 |     2     |

## [G-01] Use ++i instead of i++ to increment

The reason behind this is in way ++i and i++ are evaluated by the compiler.

i++ returns i(its old value) before incrementing i to a new value. This means that 2 values are stored on the stack for usage whether you wish to use it or not. ++i on the other hand, evaluates the ++ operation on i (i.e it increments i) then returns i (its incremented value) which means that only one item needs to be stored on the stack.

```solidity
File: contracts/Tokens/Prime/Prime.sol

189                i++;

217                    j++;

221            pendingScoreUpdates--;

225                i++;

250                i++;

345                    i++;

355                    i++;

614                i++;

636                i++;

711            totalIrrevocable++;

713            totalRevocable++;

740                i++;

745            totalIrrevocable--;

747            totalRevocable--;

766        totalIrrevocable++;

767        totalRevocable--;

819        nextScoreUpdateRoundId++;

828        if (totalScoreUpdatesRequired > 0) totalScoreUpdatesRequired--;

831            pendingScoreUpdates--;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

## [G-02] Admin functions can be payable

We can make admin specific functions payable to save gas, because the compiler won’t be checking the **callvalue** of the function.

This will also make the contract smaller and cheaper to deploy as there will be fewer **opcodes** in the creation and runtime code.

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

118    function initializeTokens(address[] calldata tokens_) external onlyOwner {


177    function setPrimeToken(address prime_) external onlyOwner {


    216    function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

## [G-03] Always use Named Returns

The solidity compiler outputs more efficient code when the variable is declared in the return statement. There seem to be very few exceptions to this in practice, so if you see an anonymous return, you should test it with a named return instead to determine which case is most efficient.

#### For-Exmaple

```solidity

contract AnonymousReturn {
    function myFunc1(uint256 x, uint256 y) external pure returns (uint256) {
        require(x > 0);
        require(y > 0);

        return x * y;
    }
}

// More Efficient Code
contract NamedReturn {
    function myFunc2(uint256 x, uint256 y) external pure returns (uint256 z) {
        require(x > 0);
        require(y > 0);

        z = x * y;
    }
}
```

```solidity
File: contracts/Tokens/Prime/Prime.sol

//  @audit remove the return statement
193        return pendingInterests;


434        return _claimInterest(vToken, msg.sender);

444        return _claimInterest(vToken, user);

470        return allMarkets;

479        if (stakedAt[user] == 0) return STAKING_PERIOD;

514        return _calculateUserAPR(market, supply, borrow, cappedSupply, cappedBorrow, userScore, totalScore);

547        return _calculateUserAPR(market, supply, borrow, cappedSupply, cappedBorrow, userScore, totalScore);

600        return _interestAccrued(vToken, user);

663        return Scores.calculateScore(xvsBalanceForScore, capital, alphaNumerator, alphaDenominator);

696        return amount;

846        return (xvs - pendingWithdrawals);

896        return ((supply + borrow), supply, borrow);

922        return (index * score) / EXP_SCALE;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

```solidity
File: Tokens/Prime/PrimeLiquidityProvider.sol

238            return distributionSpeed;

277        return block.number;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

```solidity
File: contracts/Tokens/Prime/libs/FixedMath.sol

25        return (n.toInt256() * FixedMath0x.FIXED_1) / int256(d.toInt256());

37        return uint256((u.toInt256() * FixedMath0x.FIXED_1) / f);

49        return uint256((u.toInt256() * f) / FixedMath0x.FIXED_1);

54        return FixedMath0x.ln(x);

59        return FixedMath0x.exp(x);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol

## [G-04] Make the variable outside the loop to save gas

When you declare a variable inside a loop the variable is reinitialized and assigned memory or storage space during each iteration of the loop. This can use more gas, leading to higher transaction costs. To save gas, it's often recommended to declare the variable outside the loop so that it is only initialized once and reused throughout the loop.

```solidity
File: contracts/Tokens/Prime/Prime.sol

179            address market = _allMarkets[i];
180            uint256 interestAccrued = getInterestAccrued(market, user);
181            uint256 accrued = interests[market][user].accrued;


205            address user = users[i];

212                address market = _allMarkets[j];

626            address market = _allMarkets[i];

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

## [G-05] Revert() statements should be used sorted from cheapest to most expensive

### `_primeLiquidityProvider` this check should came before the `_comptroller` and `_oracle`

```solidity
File: contracts/Tokens/Prime/Prime.sol

143        if (_xvsVault == address(0)) revert InvalidAddress();
144        if (_xvsVaultRewardToken == address(0)) revert InvalidAddress();
145        if (_protocolShareReserve == address(0)) revert InvalidAddress();
146        if (_comptroller == address(0)) revert InvalidAddress();
147        if (_oracle == address(0)) revert InvalidAddress();
148        if (_primeLiquidityProvider == address(0)) revert InvalidAddress();



// @audit first check asset for address(0)
456        address vToken = vTokenForAsset[asset];
457        if (vToken == address(0)) revert MarketNotSupported();
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

## [G-06] Should use arguments instead of the state variables

```solidity
File: contracts/Tokens/Prime/Prime.sol

// @audit alphaNumerator, alphaDenominator is state variable first cache then use in emit
241        emit AlphaUpdated(alphaNumerator, alphaDenominator, _alphaNumerator, _alphaDenominator);


462        emit UpdatedAssetsState(comptroller, asset);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

180        emit PrimeTokenUpdated(prime, prime_);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

## [G-07] Amounts should be checked for `0` before calling a transfer

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

204        IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L204

## [G-08] Use bitmaps instead of bools when a significant amount of booleans are used

A common pattern, especially in airdrops, is to mark an address as “already used” when claiming the airdrop or NFT mint.

However, since it only takes one bit to store this information, and each slot is 256 bits, that means one can store a 256 flags/booleans with one storage slot.

```solidity
File: contracts/Tokens/Prime/Prime.sol

292        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);

331    function issue(bool isIrrevocable, address[] calldata users) external {

367        bool isAccountEligible = isEligible(totalStaked);

704    function _mint(bool isIrrevocable, address user) internal {

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L292

```solidity
File:  contracts/Tokens/Prime/PrimeStorage.sol

8        bool exists;
9        bool isIrrevocable;

17        bool exists;

88    mapping(uint256 => mapping(address => bool)) public isScoreUpdated;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L8

## [G-09] Using Storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to
be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File: contracts/Tokens/Prime/Prime.sol

176        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

683            address[] memory assets = new address[](1);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L176

## [G-10] Do not calculate constants

```solidity
File: contracts/Tokens/Prime/PrimeStorage.sol

34     uint256 public constant MINIMUM_STAKED_XVS = 1000 * EXP_SCALE;

37     uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

40    uint256 public constant STAKING_PERIOD = 90 * 24 * 60 * 60;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L34

## [G-11] block.number directly call instead of a function

`block.number` is a built-in global variable that represents the current block number, it's more gas-efficient to directly use block.number as a variable rather than creating a separate function to return this value.

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

276    function getBlockNumber() public view virtual returns (uint256) {
277        return block.number;
279    }
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L276-L278

## [G-12] Use do while loops instead of for loops

If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.

For-Example

```solidity

// times == 10 in both tests
contract Loop1 {
    function loop(uint256 times) public pure {
        for (uint256 i; i < times; ) {
            unchecked {
                ++i;
            }
        }
    }
}

contract Loop2 {
    function loop(uint256 times) public pure {
        if (times == 0) {
            return;
        }

        uint256 i;

        do {
            unchecked {
                ++i;
            }
        } while (i < times);
    }
}
```

```solidity
File: contracts/Tokens/Prime/Prime.sol

178        for (uint256 i = 0; i < _allMarkets.length; ) {

204        for (uint256 i = 0; i < users.length; ) {

211            for (uint256 j = 0; j < _allMarkets.length; ) {

246        for (uint256 i = 0; i < allMarkets.length; ) {

335            for (uint256 i = 0; i < users.length; ) {

349            for (uint256 i = 0; i < users.length; ) {

609        for (uint256 i = 0; i < _allMarkets.length; ) {

625        for (uint256 i = 0; i < _allMarkets.length; ) {

730        for (uint256 i = 0; i < _allMarkets.length; ) {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L204

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

103        for (uint256 i; i < numTokens; ) {

119        for (uint256 i; i < tokens_.length; ) {

161        for (uint256 i; i < numTokens; ) {

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L161

## [G-13] Use assembly to perform efficient back-to-back calls

Use assembly to reuse memory space when making more than one external call.
An operation that causes the solidity compiler to expand memory is making external calls. When making external calls the compiler has to encode the function signature of the function it wishes to call on the external contract alongside it’s arguments in memory. As we know, solidity does not clear or reuse memory memory so it’ll have to store these data in the next free memory pointer which expands memory further.

With inline assembly, we can either use the scratch space and free memory pointer offset to store this data (as above) if the function arguments do not take up more than 96 bytes in memory. Better still, if we are making more than one external call we can reuse the same memory space as the first calls to store the new arguments in memory without expanding memory unnecessarily. Solidity in this scenario would expand memory by as much as the returned data length is. This is because the returned data is stored in memory (in most cases). If the return data is less than 96 bytes, we can use the scratch space to store it to prevent expanding memory.

See the example below;

```solidity
contract Called {
    function add(uint256 a, uint256 b) external pure returns (uint256) {
        return a + b;
    }
}

contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns (uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}

contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns (uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}

```

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

```solidity
File: contracts/Tokens/Prime/Prime.sol

498        uint256 borrow = vToken.borrowBalanceStored(user);
499        uint256 exchangeRate = vToken.exchangeRateStored();
500        uint256 balanceOfAccount = vToken.balanceOf(user);


561                uint256 totalIncomeUnreleased = IProtocolShareReserve(protocolShareReserve).getUnreleasedFunds(


570        _primeLiquidityProvider.accrueTokens(underlying);
571        uint256 totalAccruedInPLP = _primeLiquidityProvider.tokenAmountAccrued(underlying);





651        uint256 borrow = vToken.borrowBalanceStored(user);
652        uint256 exchangeRate = vToken.exchangeRateStored();
653        uint256 balanceOfAccount = vToken.balanceOf(user);


656        address xvsToken = IXVSVault(xvsVault).xvsAddress();
657        oracle.updateAssetPrice(xvsToken);
658        oracle.updatePrice(market);




685            IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
686            if (amount > asset.balanceOf(address(this))) {
687                IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));





880        uint256 xvsPrice = oracle.getPrice(xvsToken);

884        uint256 tokenPrice = oracle.getUnderlyingPrice(market);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L498-L500

## [G-14] Ternary operation is cheaper than if-else statement

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

```solidity
File: contracts/Tokens/Prime/Prime.sol


421        if (paused()) {
422            _unpause();
423        } else {
424            _pause();
425        }




482        if (totalTimeStaked < STAKING_PERIOD) {
483            return STAKING_PERIOD - totalTimeStaked;
484        } else {
485            return 0;
486        }



855        if (xvs > MAXIMUM_XVS_CAP) {
856            return MAXIMUM_XVS_CAP;
857        } else {
858            return xvs;
859        }




931        if (vToken == VBNB) {
932            return WBNB;
933        } else {
934            return IVToken(vToken).underlying();
935        }
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L931-L935

```diff
-421        if (paused()) {
-422            _unpause();
-423        } else {
-424            _pause();
-425        }

+   paused() ? _unpause() : _pause();




-482        if (totalTimeStaked < STAKING_PERIOD) {
-483            return STAKING_PERIOD - totalTimeStaked;
-484        } else {
-485            return 0;
-486        }

+   return totalTimeStaked < STAKING_PERIOD ? STAKING_PERIOD - totalTimeStaked : 0;




-855        if (xvs > MAXIMUM_XVS_CAP) {
-856            return MAXIMUM_XVS_CAP;
-857        } else {
-858            return xvs;
-859        }

+   return xvs > MAXIMUM_XVS_CAP ? MAXIMUM_XVS_CAP : xvs;




-931        if (vToken == VBNB) {
-932            return WBNB;
-933        } else {
-934            return IVToken(vToken).underlying();
-935        }

+       return (vToken == VBNB) ? WBNB : IVToken(vToken).underlying();
```

## [G-15] Short-circuit booleans

In Solidity, when you evaluate a boolean expression (e.g the || (logical or) or && (logical and) operators), in the case of || the second expression will only be evaluated if the first expression evaluates to false and in the case of && the second expression will only be evaluated if the first expression evaluates to true. This is called short-circuiting.

For example, the expression require(msg.sender == owner || msg.sender == manager) will pass if the first expression msg.sender == owner evaluates to true. The second expression msg.sender == manager will not be evaluated at all.

However, if the first expression msg.sender == owner evaluates to false, the second expression msg.sender == manager will be evaluated to determine whether the overall expression is true or false. Here, by checking the condition that is most likely to pass firstly, we can avoid checking the second condition thereby saving gas in majority of successful calls.

This is similar for the expression require(msg.sender == owner && msg.sender == manager). If the first expression msg.sender == owner evaluates to false, the second expression msg.sender == manager will not be evaluated because the overall expression cannot be true. For the overall statement to be true, both side of the expression must evaluate to true. Here, by checking the condition that is most likely to fail firstly, we can avoid checking the second condition thereby saving gas in majority of call reverts.

Short-circuiting is useful and it’s recommended to place the less expensive expression first, as the more costly one might be bypassed. If the second expression is more important than the first, it might be worth reversing their order so that the cheaper one gets evaluated first.

```solidity
File: contracts/Tokens/Prime/Prime.sol

318        if (_irrevocableLimit < totalIrrevocable || _revocableLimit < totalRevocable) revert InvalidLimit();


337                if (userToken.exists && !userToken.isIrrevocable) {

369        if (tokens[user].exists && !isAccountEligible) {

375        } else if (!isAccountEligible && !tokens[user].exists && stakedAt[user] > 0) {

377        } else if (stakedAt[user] == 0 && isAccountEligible && !tokens[user].exists) {

379        } else if (tokens[user].exists && isAccountEligible) {

780        if (!markets[vToken].exists || !tokens[user].exists) {

795        if (!markets[market].exists || !tokens[user].exists) {

810        if (_alphaDenominator == 0 || _alphaNumerator > _alphaDenominator) {

830        if (pendingScoreUpdates > 0 && !isScoreUpdated[nextScoreUpdateRoundId][user]) {

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L318

## [G-16] Unnecessary casting as variable is already of the same type

```solidity
File: contracts/Tokens/Prime/Prime.sol

// @audit vToken is already the address type it's unnecessary cast to address
511            address(vToken)


// @audit unnecessary casting of asset
684            assets[0] = address(asset);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L511

## [G-17] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

https://twitter.com/transmissions11/status/1518507047943245824

```solidity
File: contracts/Tokens/Prime/Prime.sol

564            address(this),

682        if (amount > asset.balanceOf(address(this))) {

686            if (amount > asset.balanceOf(address(this))) {

960                address(this),

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol


217        uint256 balance = token_.balanceOf(address(this));

234        uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

259            uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol

## [G-18] Using mappings instead of arrays to avoid length checks

When storing a list or group of items that you wish to organize in a specific order and fetch with a fixed key/index, it’s common practice to use an array data structure. This works well, but did you know that a trick can be implemented to save 2,000+ gas on each read using a mapping?

See the example below

```solidity
/// get(0) gas cost: 4860
contract Array {
    uint256[] a;

    constructor() {
        a.push() = 1;
        a.push() = 2;
        a.push() = 3;
    }

    function get(uint256 index) external view returns (uint256) {
        return a[index];
    }
}

/// get(0) gas cost: 2758
contract Mapping {
    mapping(uint256 => uint256) a;

    constructor() {
        a[0] = 1;
        a[1] = 2;
        a[2] = 3;
    }

    function get(uint256 index) external view returns (uint256) {
        return a[index];
    }
}

```

Just by using a mapping, we get a gas saving of 2102 gas. Why? Under the hood when you read the value of an index of an array, solidity adds bytecode that checks that you are reading from a valid index (i.e an index strictly less than the length of the array) else it reverts with a panic error (Panic(0x32) to be precise). This prevents from reading unallocated or worse, allocated storage/memory locations.

Due to the way mappings are (simply a key => value pair), no check like that exists and we are able to read from the a storage slot directly. It’s important to note that when using mappings in this manner, your code should ensure that you are not reading an out of bound index of your canonical array.

```solidity
File: contracts/Tokens/Prime/Prime.sol

175        address[] storage _allMarkets = allMarkets;

210        address[] storage _allMarkets = allMarkets;

608        address[] storage _allMarkets = allMarkets;

624        address[] storage _allMarkets = allMarkets;

728        address[] storage _allMarkets = allMarkets;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol

```solidity
File: contracts/Tokens/Prime/PrimeStorage.sol

70    address[] internal allMarkets;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeStorage.sol#L70

## [G-19] Using delete statement can save gas

```solidity
File: contracts/Tokens/Prime/Prime.sol

156        nextScoreUpdateRoundId = 0;

298        markets[vToken].sumOfMembersScore = 0;

376            stakedAt[user] = 0;

401        stakedAt[msg.sender] = 0;

460        unreleasedPSRIncome[_getUnderlying(address(market))] = 0;

677        interests[vToken][user].accrued = 0;

688                unreleasedPLPIncome[underlying] = 0;

736            interests[_allMarkets[i]][user].score = 0;

737            interests[_allMarkets[i]][user].rewardIndex = 0;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L156

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

200        tokenAmountAccrued[token_] = 0;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L200

## [G-20] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
File: contracts/Tokens/Prime/Prime.sol

176        PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);

683            address[] memory assets = new address[](1);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L176

## [G-21] Use v4.9.0 OpenZeppelin contracts

The upcoming v4.9.0 version of OpenZeppelin provides many small gas optimizations.

https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0

```javascript
File: main/package.json

34    "@openzeppelin/contracts": "^4.8.3",
35    "@openzeppelin/contracts-upgradeable": "^4.8.0",

```

https://github.com/code-423n4/2023-09-venus/blob/main/package.json#L34

## [G-22] Mappings used within a function more than once should be cached to save gas

```solidity
File: contracts/Tokens/Prime/Prime.sol

271            markets[market].supplyMultiplier,
272            markets[market].borrowMultiplier,

276        markets[market].supplyMultiplier = supplyMultiplier;
277        markets[market].borrowMultiplier = borrowMultiplier;




369        if (tokens[user].exists && !isAccountEligible) {

375        } else if (!isAccountEligible && !tokens[user].exists && stakedAt[user] > 0) {

377        } else if (stakedAt[user] == 0 && isAccountEligible && !tokens[user].exists) {

379        } else if (tokens[user].exists && isAccountEligible) {






398        if (stakedAt[msg.sender] == 0) revert IneligibleToClaim();
399        if (block.timestamp - stakedAt[msg.sender] < STAKING_PERIOD) revert WaitMoreTime();

401        stakedAt[msg.sender] = 0;



580        unreleasedPSRIncome[underlying] = totalIncomeUnreleased;
581        unreleasedPLPIncome[underlying] = totalAccruedInPLP;



733            markets[_allMarkets[i]].sumOfMembersScore =
734                markets[_allMarkets[i]].sumOfMembersScore -
735                interests[_allMarkets[i]][user].score;
736            interests[_allMarkets[i]][user].score = 0;
737            interests[_allMarkets[i]][user].rewardIndex = 0;
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L271

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol


199        uint256 accruedAmount = tokenAmountAccrued[token_];
200        tokenAmountAccrued[token_] = 0;



261            uint256 balanceDiff = balance - tokenAmountAccrued[token_];
266                tokenAmountAccrued[token_] += tokenAccrued;


315        if (tokenDistributionSpeeds[token_] != distributionSpeed_) {
322            tokenDistributionSpeeds[token_] = distributionSpeed_;



255        uint256 deltaBlocks = blockNumber - lastAccruedBlock[token_];
270            lastAccruedBlock[token_] = blockNumber;



289        uint256 initializedBlock = lastAccruedBlock[token_];
298        lastAccruedBlock[token_] = blockNumber;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L315

## [G-23] Divisions can be unchecked to save gas from previous check

```solidity
File: contracts/Tokens/Prime/Prime.sol

584        if (markets[vToken].sumOfMembersScore > 0) {
585            delta = ((distributionIncome * EXP_SCALE) / markets[vToken].sumOfMembersScore); // @audit there should use unchecked because no overflow from previous  if statements
586        }



889            supply = supplyUSD > 0 ? (supply * supplyCapUSD) / supplyUSD : 0;


893            borrow = borrowUSD > 0 ? (borrow * borrowCapUSD) / borrowUSD : 0;

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L584-L586

## [G-24] Initializers can be marked as payable to save deployment gas

```solidity
File: contracts/Tokens/Prime/Prime.sol



130    function initialize(
...

142    ) external virtual initializer {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L142

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

90    function initialize(
91        address accessControlManager_,
92        address[] calldata tokens_,
93        uint256[] calldata distributionSpeeds_
94    ) external initializer {
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L90-L94

## [G-25] Consider using alternatives to OpenZeppelin

OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.

Two examples of such alternatives are Solmate and Solady.

Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.

```solidity
File: contracts/Tokens/Prime/Prime.sol

4   import { SafeERC20Upgradeable, IERC20Upgradeable } from "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";


7   import { PausableUpgradeable } from "@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L4

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

4   import { SafeERC20Upgradeable, IERC20Upgradeable } from "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";


6   import { PausableUpgradeable } from "@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L4

```solidity
File: contracts/Tokens/Prime/libs/Scores.sol

5   import { SafeCastUpgradeable } from "@openzeppelin/contracts-upgradeable/utils/math/SafeCastUpgradeable.sol";

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/Scores.sol#L5

```solidity
File: contracts/Tokens/Prime/libs/FixedMath.sol

6   import { SafeCastUpgradeable } from "@openzeppelin/contracts-upgradeable/utils/math/SafeCastUpgradeable.sol";
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L6

## [G-26] Using assembly to revert with an error message

When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Here’s an example;

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num) external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num) external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}

```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

```solidity
File: contracts/Tokens/Prime/Prime.sol

104        if (_wbnb == address(0)) revert InvalidAddress();
105        if (_vbnb == address(0)) revert InvalidAddress();
106        if (_blocksPerYear == 0) revert InvalidBlocksPerYear();

143        if (_xvsVault == address(0)) revert InvalidAddress();
144        if (_xvsVaultRewardToken == address(0)) revert InvalidAddress();
145        if (_protocolShareReserve == address(0)) revert InvalidAddress();
146        if (_comptroller == address(0)) revert InvalidAddress();
147        if (_oracle == address(0)) revert InvalidAddress();
148        if (_primeLiquidityProvider == address(0)) revert InvalidAddress();

201        if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
202        if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

207            if (!tokens[user].exists) revert UserHasNoPrimeToken();

265        if (!markets[market].exists) revert MarketNotSupported();

290        if (markets[vToken].exists) revert MarketAlreadyExists();

293        if (!isMarketExist) revert InvalidVToken();

318        if (_irrevocableLimit < totalIrrevocable || _revocableLimit < totalRevocable) revert InvalidLimit();

398        if (stakedAt[msg.sender] == 0) revert IneligibleToClaim();
399        if (block.timestamp - stakedAt[msg.sender] < STAKING_PERIOD) revert WaitMoreTime();

453        if (msg.sender != protocolShareReserve) revert InvalidCaller();
454        if (comptroller != _comptroller) revert InvalidComptroller();

457        if (vToken == address(0)) revert MarketNotSupported();

555        if (!markets[vToken].exists) revert MarketNotSupported();

705        if (tokens[user].exists) revert IneligibleToClaim();

716        if (totalIrrevocable > irrevocableLimit || totalRevocable > revocableLimit) revert InvalidLimit();

726        if (!tokens[user].exists) revert UserHasNoPrimeToken();

769        if (totalIrrevocable > irrevocableLimit) revert InvalidLimit();

811            revert InvalidAlphaArguments();
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L811

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

100            revert InvalidArguments();

158            revert InvalidArguments();

193        if (msg.sender != prime) revert InvalidCaller();

195            revert FundsTransferIsPaused();

219            revert InsufficientBalance(amount_, balance);

292            revert TokenAlreadyInitialized(token_);

312            revert InvalidDistributionSpeed(distributionSpeed_, MAX_DISTRIBUTION_SPEED);

336            revert TokenNotInitialized(token_);

346            revert InvalidArguments();

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L100

```solidity
File: contracts/Tokens/Prime/libs/FixedMath.sol


23        if (d.toInt256() < n.toInt256()) revert InvalidFraction(n, d);

35        if (f < 0) revert InvalidFixedPoint();

47        if (f < 0) revert InvalidFixedPoint();

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L23

## [G-27] Split revert statements

Similar to splitting require statements, you will usually save some gas by not having a boolean operator in the if statement.

```solidity
contract CustomErrorBoolLessEfficient {
error BadValue();

    function requireGood(uint256 x) external pure {
        if (x < 10 || x > 20) {
            revert BadValue();
        }
    }

}

contract CustomErrorBoolEfficient {
error TooLow();
error TooHigh();

    function requireGood(uint256 x) external pure {
        if (x < 10) {
            revert TooLow();
        }
        if (x > 20) {
            revert TooHigh();
        }
    }

}
```

```solidity
File: contracts/Tokens/Prime/Prime.sol

318        if (_irrevocableLimit < totalIrrevocable || _revocableLimit < totalRevocable) revert InvalidLimit();


716        if (totalIrrevocable > irrevocableLimit || totalRevocable > revocableLimit) revert InvalidLimit();

810        if (_alphaDenominator == 0 || _alphaNumerator > _alphaDenominator) {
811            revert InvalidAlphaArguments();
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L318

## [G-28] Make event parameters indexed when possible

Events are used to emit information about state changes in a contract. When defining an event, it's important to consider the gas cost of emitting the event, as well as the efficiency of searching for events in the Ethereum blockchain.

Making event parameters indexed can help reduce the gas cost of emitting events and improve the efficiency of searching for events. When an event parameter is marked as indexed, its value is stored in a separate data structure called the event topic, which allows

```solidity
File: contracts/Tokens/Prime/Prime.sol

91    event InterestClaimed(address indexed user, address indexed market, uint256 amount);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L91

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

36    event PrimeTokenUpdated(address oldPrimeToken, address newPrimeToken);

```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L36
