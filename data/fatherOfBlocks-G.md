**contracts/Tokens/Prime/Prime.sol**
- L905/906/909 - Can be simplified to "return amount >= MINIMUM_STAKED_XVS;"


**contracts/Tokens/Prime/PrimeLiquidityProvider.sol**
- L15 - The EXP_SCALE constant is created, but it is never used, therefore it should be removed.

- L48/51/54 - The TokenInitialBalanceUpdated, FundsTransferResumed, FundsTransferPaused event is created, but they are never used, therefore they should be eliminated.


**contracts/Tokens/Prime/libs/FixedMath0x.sol**
- L36 - The UnsignedValueTooLarge error is created, but it is never used, therefore it should be removed.