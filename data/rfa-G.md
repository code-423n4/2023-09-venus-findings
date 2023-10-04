## [G-01] Initialization new markets.value with default value

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L295
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L298

```solidity
if (markets[vToken].exists) revert MarketAlreadyExists();

        bool isMarketExist = InterfaceComptroller(comptroller).markets(vToken);
        if (!isMarketExist) revert InvalidVToken();

        markets[vToken].rewardIndex = 0;
        markets[vToken].supplyMultiplier = supplyMultiplier;
        markets[vToken].borrowMultiplier = borrowMultiplier;
        markets[vToken].sumOfMembersScore = 0;
```

```
|  Prime                   ·  addMarket                     ·     190892  ·     230704  ·     207156  ·            7  ·          -  │ //before
···························|································|·············|·············|·············|···············|··············
|  Prime                   ·  addMarket                     ·     186462  ·     226274  ·     202726  ·            7  ·          -  │ //after
```

At LOC (290)[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L290] the `addMarket()` function has checked whether the marked is existed or not. The `markets.existed` will always evaluated as false, therefore it's unnecessary to initialize the value of `markets.rewardIndex` and `markets.sumOfMemberScore` with 0

### Recommended Mitigation step

