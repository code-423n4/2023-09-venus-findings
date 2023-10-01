# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Cache array length outside of loop | 11 |
| [GAS-2](#GAS-2) | Use calldata instead of memory for function arguments that do not get mutated | 2 |
| [GAS-3](#GAS-3) | boolean literals comparisan should be avoided | 5 |
| [GAS-4](#GAS-4) | Don't initialize variables with default value | 20 |
| [GAS-5](#GAS-5) | Long revert strings | 79 |
| [GAS-6](#GAS-6) | Use mappings over arrays | 17 |
| [GAS-7](#GAS-7) | Use shift Right/Left instead of division/multiplication if possible | 2 |
### <a name="GAS-1"></a>[GAS-1] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (11)*:
```solidity
File: Prime/Prime.sol

178:         for (uint256 i = 0; i < _allMarkets.length; ) {

204:         for (uint256 i = 0; i < users.length; ) {

211:             for (uint256 j = 0; j < _allMarkets.length; ) {

246:         for (uint256 i = 0; i < allMarkets.length; ) {

335:             for (uint256 i = 0; i < users.length; ) {

349:             for (uint256 i = 0; i < users.length; ) {

609:         for (uint256 i = 0; i < _allMarkets.length; ) {

625:         for (uint256 i = 0; i < _allMarkets.length; ) {

730:         for (uint256 i = 0; i < _allMarkets.length; ) {

```

```solidity
File: Prime/PrimeLiquidityProvider.sol

119:         for (uint256 i; i < tokens_.length; ) {

```

```solidity
File: VAI/VAIController.sol

426:         for (i = 0; i < enteredMarkets.length; i++) {

```

### <a name="GAS-2"></a>[GAS-2] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (2)*:
```solidity
File: Prime/Interfaces/IProtocolShareReserve.sol

10:     function releaseFunds(address comptroller, address[] memory assets) external;

```

```solidity
File: Prime/Prime.sol

200:     function updateScores(address[] memory users) external {

```

### <a name="GAS-3"></a>[GAS-3] boolean literals comparisan should be avoided

*Instances (5)*:
```solidity
File: VAI/VAIController.sol

228:         require(success == true, "failed to transfer VAI fee");

```

```solidity
File: VRT/VRTConverter.sol

58:         require(initialized == false, "VRTConverter is already initialized");

105:         require(initialized == true, "VRTConverter is not initialized");

```

```solidity
File: XVS/XVSVesting.sol

47:         require(initialized == false, "XVSVesting is already initialized");

56:         require(initialized == true, "XVSVesting is not initialized");

```

### <a name="GAS-4"></a>[GAS-4] Don't initialize variables with default value

*Instances (20)*:
```solidity
File: Prime/Prime.sol

178:         for (uint256 i = 0; i < _allMarkets.length; ) {

204:         for (uint256 i = 0; i < users.length; ) {

211:             for (uint256 j = 0; j < _allMarkets.length; ) {

246:         for (uint256 i = 0; i < allMarkets.length; ) {

335:             for (uint256 i = 0; i < users.length; ) {

349:             for (uint256 i = 0; i < users.length; ) {

609:         for (uint256 i = 0; i < _allMarkets.length; ) {

625:         for (uint256 i = 0; i < _allMarkets.length; ) {

677:         interests[vToken][user].accrued = 0;

730:         for (uint256 i = 0; i < _allMarkets.length; ) {

736:             interests[_allMarkets[i]][user].score = 0;

737:             interests[_allMarkets[i]][user].rewardIndex = 0;

```

```solidity
File: Prime/libs/FixedMath0x.sol

46:     int256 private constant EXP_MAX_VAL = 0;

```

```solidity
File: VAI/VAIController.sol

481:         uint256 repayAmount = 0;

```

```solidity
File: VRT/VRT.sol

215:         uint32 lower = 0;

```

```solidity
File: VTokens/VToken.sol

558:         uint startingAllowance = 0;

```

```solidity
File: XVS/XVS.sol

215:         uint32 lower = 0;

```

```solidity
File: XVS/XVSVesting.sol

119:         uint256 totalWithdrawableAmount = 0;

121:         for (uint i = 0; i < vestingCount; ++i) {

160:         for (uint i = 0; i < vestingCount; i++) {

```

### <a name="GAS-5"></a>[GAS-5] Long revert strings

*Instances (79)*:
```solidity
File: VAI/VAIController.sol

89:         require(msg.sender == unitroller.admin(), "only unitroller admin can change brains");

151:             require(vars.mathErr == MathError.NO_ERROR, "VAI_MINT_AMOUNT_CALCULATION_FAILED");

234:         require(mErr == MathError.NO_ERROR, "VAI_BURN_AMOUNT_CALCULATION_FAILED");

237:         require(mErr == MathError.NO_ERROR, "VAI_BURN_AMOUNT_CALCULATION_FAILED");

240:         require(mErr == MathError.NO_ERROR, "VAI_BURN_AMOUNT_CALCULATION_FAILED");

493:         require(vars.mErr == MathError.NO_ERROR, "VAI_MINT_AMOUNT_CALCULATION_FAILED");

496:         require(vars.mErr == MathError.NO_ERROR, "VAI_MINT_AMOUNT_CALCULATION_FAILED");

545:                     require(mErr == MathError.NO_ERROR, "VAI_REPAY_RATE_CALCULATION_FAILED");

548:                     require(mErr == MathError.NO_ERROR, "VAI_REPAY_RATE_CALCULATION_FAILED");

551:                     require(mErr == MathError.NO_ERROR, "VAI_REPAY_RATE_CALCULATION_FAILED");

554:                     require(mErr == MathError.NO_ERROR, "VAI_REPAY_RATE_CALCULATION_FAILED");

575:         require(mErr == MathError.NO_ERROR, "VAI_REPAY_RATE_CALCULATION_FAILED");

605:         require(mErr == MathError.NO_ERROR, "VAI_TOTAL_REPAY_AMOUNT_CALCULATION_FAILED");

608:         require(mErr == MathError.NO_ERROR, "VAI_TOTAL_REPAY_AMOUNT_CALCULATION_FAILED");

611:         require(mErr == MathError.NO_ERROR, "VAI_TOTAL_REPAY_AMOUNT_CALCULATION_FAILED");

614:         require(mErr == MathError.NO_ERROR, "VAI_TOTAL_REPAY_AMOUNT_CALCULATION_FAILED");

617:         require(mErr == MathError.NO_ERROR, "VAI_TOTAL_REPAY_AMOUNT_CALCULATION_FAILED");

634:         require(mErr == MathError.NO_ERROR, "VAI_BURN_AMOUNT_CALCULATION_FAILED");

637:         require(mErr == MathError.NO_ERROR, "VAI_BURN_AMOUNT_CALCULATION_FAILED");

645:             require(mErr == MathError.NO_ERROR, "VAI_BURN_AMOUNT_CALCULATION_FAILED");

657:             require(mErr == MathError.NO_ERROR, "VAI_MINTED_AMOUNT_CALCULATION_FAILED");

660:             require(mErr == MathError.NO_ERROR, "VAI_BURN_AMOUNT_CALCULATION_FAILED");

663:             require(mErr == MathError.NO_ERROR, "VAI_BURN_AMOUNT_CALCULATION_FAILED");

666:             require(mErr == MathError.NO_ERROR, "VAI_CURRENT_INTEREST_AMOUNT_CALCULATION_FAILED");

669:             require(mErr == MathError.NO_ERROR, "VAI_CURRENT_INTEREST_AMOUNT_CALCULATION_FAILED");

672:             require(mErr == MathError.NO_ERROR, "VAI_PAST_INTEREST_CALCULATION_FAILED");

675:             require(mErr == MathError.NO_ERROR, "VAI_PAST_INTEREST_CALCULATION_FAILED");

```

```solidity
File: VRT/VRT.sol

174:         require(signatory != address(0), "VRT::delegateBySig: invalid signature");

175:         require(nonce == nonces[signatory]++, "VRT::delegateBySig: invalid nonce");

176:         require(now <= expiry, "VRT::delegateBySig: signature expired");

198:         require(blockNumber < block.number, "VRT::getPriorVotes: not yet determined");

242:         require(src != address(0), "VRT::_transferTokens: cannot transfer from the zero address");

243:         require(dst != address(0), "VRT::_transferTokens: cannot transfer to the zero address");

```

```solidity
File: VRT/VRTConverter.sol

57:         require(msg.sender == admin, "only admin may initialize the VRTConverter");

58:         require(initialized == false, "VRTConverter is already initialized");

69:         require(_conversionStartTime >= block.timestamp, "conversionStartTime must be time in the future");

98:         require(msg.sender == admin, "only admin may initialize the Vault");

158:         require(msg.sender == vrtConverterProxy.admin(), "only proxy admin can change brains");

```

```solidity
File: VRT/VRTConverterProxy.sol

64:         require(msg.sender == admin, "VRTConverterProxy::_setImplementation: admin only");

65:         require(implementation_ != address(0), "VRTConverterProxy::_setImplementation: invalid implementation address");

94:         require(msg.sender == admin, "Only admin can set Pending Implementation");

136:         require(newPendingAdmin != pendingAdmin, "New pendingAdmin can not be same as the previous one");

155:         require(msg.sender == pendingAdmin, "only address marked as pendingAdmin can accept as Admin");

```

```solidity
File: VTokens/VBep20Delegate.sol

29:         require(msg.sender == admin, "only the admin may call _becomeImplementation");

41:         require(msg.sender == admin, "only the admin may call _resignImplementation");

```

```solidity
File: VTokens/VBep20Delegator.sol

66:         require(msg.value == 0, "VBep20Delegator:fallback: cannot send value to fallback");

403:         require(msg.sender == admin, "VBep20Delegator::_setImplementation: Caller must be admin");

```

```solidity
File: VTokens/VToken.sol

322:         require(msg.sender == admin, "only admin may initialize the market");

323:         require(accrualBlockNumber == 0 && borrowIndex == 0, "market may only be initialized once");

327:         require(initialExchangeRateMantissa > 0, "initial exchange rate must be greater than zero.");

339:         require(err == uint(Error.NO_ERROR), "setting interest rate model failed");

521:         require(err == MathError.NO_ERROR, "exchangeRateStored: exchangeRateStoredInternal failed");

532:         require(err == MathError.NO_ERROR, "borrowBalanceStored: borrowBalanceStoredInternal failed");

679:         require(vars.mathErr == MathError.NO_ERROR, "MINT_NEW_TOTAL_SUPPLY_CALCULATION_FAILED");

682:         require(vars.mathErr == MathError.NO_ERROR, "MINT_NEW_ACCOUNT_BALANCE_CALCULATION_FAILED");

774:         require(vars.mathErr == MathError.NO_ERROR, "MINT_NEW_TOTAL_SUPPLY_CALCULATION_FAILED");

777:         require(vars.mathErr == MathError.NO_ERROR, "MINT_NEW_ACCOUNT_BALANCE_CALCULATION_FAILED");

835:         require(redeemTokensIn == 0 || redeemAmountIn == 0, "one of redeemTokensIn or redeemAmountIn must be zero");

1156:         require(vars.mathErr == MathError.NO_ERROR, "REPAY_BORROW_NEW_ACCOUNT_BORROW_BALANCE_CALCULATION_FAILED");

1159:         require(vars.mathErr == MathError.NO_ERROR, "REPAY_BORROW_NEW_TOTAL_BALANCE_CALCULATION_FAILED");

1273:         require(amountSeizeError == uint(Error.NO_ERROR), "LIQUIDATE_COMPTROLLER_CALCULATE_AMOUNT_SEIZE_FAILED");

```

```solidity
File: XVS/XVS.sol

174:         require(signatory != address(0), "XVS::delegateBySig: invalid signature");

175:         require(nonce == nonces[signatory]++, "XVS::delegateBySig: invalid nonce");

176:         require(now <= expiry, "XVS::delegateBySig: signature expired");

198:         require(blockNumber < block.number, "XVS::getPriorVotes: not yet determined");

242:         require(src != address(0), "XVS::_transferTokens: cannot transfer from the zero address");

243:         require(dst != address(0), "XVS::_transferTokens: cannot transfer to the zero address");

```

```solidity
File: XVS/XVSVesting.sol

46:         require(msg.sender == admin, "only admin may initialize the XVSVesting");

47:         require(initialized == false, "XVSVesting is already initialized");

66:         require(msg.sender == admin, "only admin may initialize the Vault");

67:         require(_vrtConversionAddress != address(0), "vrtConversionAddress cannot be Zero");

78:         require(msg.sender == vrtConversionAddress, "only VRTConversion Address can call the function");

83:         require(vestings[recipient].length > 0, "recipient doesnot have any vestingRecord");

223:         require(msg.sender == xvsVestingProxy.admin(), "only proxy admin can change brains");

```

```solidity
File: XVS/XVSVestingProxy.sol

50:         require(msg.sender == admin, "XVSVestingProxy::_setImplementation: admin only");

51:         require(implementation_ != address(0), "XVSVestingProxy::_setImplementation: invalid implementation address");

80:         require(msg.sender == admin, "Only admin can set Pending Implementation");

120:         require(newPendingAdmin != pendingAdmin, "New pendingAdmin can not be same as the previous one");

138:         require(msg.sender == pendingAdmin, "only address marked as pendingAdmin can accept as Admin");

```

### <a name="GAS-6"></a>[GAS-6] Use mappings over arrays
Arrays uses more gas than mappings. Consider using mappings whenever possible.

*Instances (17)*:
```solidity
File: Prime/Interfaces/IProtocolShareReserve.sol

10:     function releaseFunds(address comptroller, address[] memory assets) external;

```

```solidity
File: Prime/Prime.sol

175:         address[] storage _allMarkets = allMarkets;

200:     function updateScores(address[] memory users) external {

210:             address[] storage _allMarkets = allMarkets;

331:     function issue(bool isIrrevocable, address[] calldata users) external {

332:         _checkAccessAllowed("issue(bool,address[])");

469:     function getAllMarkets() external view returns (address[] memory) {

608:         address[] storage _allMarkets = allMarkets;

624:         address[] storage _allMarkets = allMarkets;

683:             address[] memory assets = new address[](1);

683:             address[] memory assets = new address[](1);

728:         address[] storage _allMarkets = allMarkets;

```

```solidity
File: Prime/PrimeLiquidityProvider.sol

92:         address[] calldata tokens_,

118:     function initializeTokens(address[] calldata tokens_) external onlyOwner {

153:     function setTokensDistributionSpeed(address[] calldata tokens_, uint256[] calldata distributionSpeeds_) external {

154:         _checkAccessAllowed("setTokensDistributionSpeed(address[],uint256[])");

```

```solidity
File: Prime/PrimeStorage.sol

70:     address[] internal allMarkets;

```

### <a name="GAS-7"></a>[GAS-7] Use shift Right/Left instead of division/multiplication if possible

*Instances (2)*:
```solidity
File: VRT/VRT.sol

218:             uint32 center = upper - (upper - lower) / 2; // ceil, avoiding overflow

```

```solidity
File: XVS/XVS.sol

218:             uint32 center = upper - (upper - lower) / 2; // ceil, avoiding overflow

```

