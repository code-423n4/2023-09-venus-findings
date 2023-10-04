## Non Critical Issues

|       | Issue                                                                                               |
| ----- | :-------------------------------------------------------------------------------------------------- |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                                                |
| NC-2  | Missing checks for `address(0)` when assigning values to address state variables                    |
| NC-3  | GENERATE PERFECT CODE HEADERS EVERY TIME                                                            |
| NC-4  | INCONSISTENT SOLIDITY VERSIONS                                                                      |
| NC-5  | MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL                                              |
| NC-6  | DON’T WORRY ABOUT KECCAK256 COLLISIONS                                                              |
| NC-7  | LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS                                        |
| NC-8  | LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION                                               |
| NC-9  | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION                                       |
| NC-10 | MISSING FEE PARAMETER VALIDATION                                                                    |
| NC-11 | NO SAME VALUE INPUT CONTROL                                                                         |
| NC-12 | SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC                                                  |
| NC-13 | USE A MORE RECENT VERSION OF SOLIDITY                                                               |
| NC-14 | STOP USING V != 27 && V != 28 OR V == 27                                                            |
| NC-15 | IMPLEMENT SOME TYPE OF VERSION COUNTER THAT WILL BE INCREMENTED AUTOMATICALLY FOR CONTRACT UPGRADES |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

316:     function setLimit(uint256 _irrevocableLimit, uint256 _revocableLimit) external {

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

153:     function setTokensDistributionSpeed(address[] calldata tokens_, uint256[] calldata distributionSpeeds_) external {

177:     function setPrimeToken(address prime_) external onlyOwner {

310:     function _setTokenDistributionSpeed(address token_, uint256 distributionSpeed_) internal {

```

### [NC-2] Missing checks for `address(0)` when assigning values to address state variables

#### Description:

0 address control should be done in these parts;

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

181:         prime = prime_;

```

#### Recommended Mitigation Steps:

Add code like this: `if (oracle == address(0)) revert ADDRESS_ZERO();`

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/IPrime.sol

2: pragma solidity ^0.5.16;

```

```solidity
File: contracts/Tokens/Prime/Prime.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/PrimeStorage.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath.sol

4: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

4: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/Scores.sol

3: pragma solidity 0.8.13;

```

### [NC-4] INCONSISTENT SOLIDITY VERSIONS

#### Description:

The project is compiled with different versions of Solidity, which is not recommended because it can lead to undefined behaviors.

It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/IPrime.sol

2: pragma solidity ^0.5.16;

```

```solidity
File: contracts/Tokens/Prime/Prime.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/PrimeStorage.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath.sol

4: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

4: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/Scores.sol

3: pragma solidity 0.8.13;

```

### [NC-5] MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL

#### Description:

If someone wants to extend via inheritance, it might make more sense that the overridden initialize(...) function calls the internal {...}\_init function, not the parent public initialize(...) function.

External instead of public would give more sense of the initialize(...) functions to behave like a constructor (only called on deployment, so should only be called externally).

From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

130:     function initialize(

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

90:     function initialize(

```

### [NC-6] DON’T WORRY ABOUT KECCAK256 COLLISIONS

#### Description:

When cancelBid is called, the commitment is set to 0 in order to force to fail the computeCommitment comparison.In case of the result of computeCommitment returns 0, because a collision or an error, a cancelled bid will be processed as valid.It’s more resilient to use a different value as flag, b.pubKey will be safer and cheaper.

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

208:             if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;

```

### [NC-7] LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTIONS

#### Description:

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the initialize() functions:

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

130:     function initialize(

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

90:     function initialize(

```

### [NC-8] LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION

#### Description:

OUse (e.g. 1e6) rather than decimal literals (e.g. 100000), for better code readability.

Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.

https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

https://github.com/code-423n4/2022-10-holograph-findings/blob/main/data/Rolezn-Q.md#NC%E2%80%917

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/PrimeStorage.sol

37:     uint256 public constant MAXIMUM_XVS_CAP = 100000 * EXP_SCALE;

43:     uint256 internal constant MAXIMUM_BPS = 10000;

```

### [NC-9] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### Description:

https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/

[code4arena example](https://code4rena.com/reports/2022-10-thegraph#n-15--for-extended-using-for-usage-use-the-latest-pragma-version)

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/IPrime.sol

2: pragma solidity ^0.5.16;

```

```solidity
File: contracts/Tokens/Prime/Prime.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/PrimeStorage.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath.sol

4: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

4: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/Scores.sol

3: pragma solidity 0.8.13;

```

#### Recommended Mitigation Steps:

Use solidity pragma version min. 0.8.13

### [NC-10] MISSING FEE PARAMETER VALIDATION

#### Description:

Some fee parameters of functions are not checked for invalid values. Validate the parameters:

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

178:         r += z * 0x00000618fee9f800; // add y^09 * (20! / 09!)

```

### [NC-11] NO SAME VALUE INPUT CONTROL

#### Description:

IR HAS TO BE IN INPUT like address oracle should be inputAdd code like this; `if (oracle == _oracle revert ADDRESS_SAME();`

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

107:         WBNB = _wbnb;

108:         VBNB = _vbnb;

151:         alphaNumerator = _alphaNumerator;

152:         alphaDenominator = _alphaDenominator;

153:         xvsVaultRewardToken = _xvsVaultRewardToken;

154:         xvsVaultPoolId = _xvsVaultPoolId;

155:         xvsVault = _xvsVault;

157:         protocolShareReserve = _protocolShareReserve;

158:         primeLiquidityProvider = _primeLiquidityProvider;

159:         comptroller = _comptroller;

243:         alphaNumerator = _alphaNumerator;

244:         alphaDenominator = _alphaDenominator;

322:         revocableLimit = _revocableLimit;

323:         irrevocableLimit = _irrevocableLimit;

```

### [NC-12] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC

#### Description:

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### **Proof Of Concept**

```solidity
File: hardhat.config.ts:

61:	  solidity: {
    compilers: [
      {
        version: "0.5.16",
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
          outputSelection: {
            "*": {
              "*": ["storageLayout"],
            },
          },
        },
      },
      {
        version: "0.8.13",
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
          outputSelection: {
            "*": {
              "*": ["storageLayout"],
            },
          },
        },
      },
    ],
  },
```

#### Recommended Mitigation Steps

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug.

Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

### [NC-13] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/IPrime.sol

2: pragma solidity ^0.5.16;

```

```solidity
File: contracts/Tokens/Prime/Prime.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/PrimeStorage.sol

2: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath.sol

4: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

4: pragma solidity 0.8.13;

```

```solidity
File: contracts/Tokens/Prime/libs/Scores.sol

3: pragma solidity 0.8.13;

```

### [NC-14] STOP USING V != 27 && V != 28 OR V == 27 || V == 28

#### Description:

[Reference](https://twitter.com/alexberegszaszi/status/1534461421454606336?s=20&t=H0Dv3ZT2bicx00hLWJk7Fg)

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/libs/Scores.sol

44:         if (xvs == 0 || capital == 0) return 0;

```

### [NC-15] IMPLEMENT SOME TYPE OF VERSION COUNTER THAT WILL BE INCREMENTED AUTOMATICALLY FOR CONTRACT UPGRADES

#### Description:

I suggest implementing some kind of version counter that will be incremented automatically when you upgrade the contract.

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

338:                     _upgrade(users[i]);

762:     function _upgrade(address user) internal {

```

## Low Issues

|     | Issue                                                                      |
| --- | :------------------------------------------------------------------------- |
| L-1 | Initializing state-variables in proxy-based upgradeable contracts          |
| L-2 | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE                             |
| L-3 | The critical parameters in initialize(...) are not set safely              |
| L-4 | INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY                             |
| L-5 | MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS |
| L-6 | Unify return criteria                                                      |
| L-7 | TIMESTAMP DEPENDENCE                                                       |
| L-8 | USE `_SAFEMINT` INSTEAD OF `_MINT`                                         |

### [L-1] Initializing state-variables in proxy-based upgradeable contracts

#### Description:

This should be done in initializer functions and not as part of the state variable declarations in which case they won’t be set.

https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#avoid-initial-values-in-field-declarations

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

130:     function initialize(

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

90:     function initialize(

```

### [L-2] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE

#### Description:

The critical procedures should be two step process.

Changing critical addresses in contracts should be a two-step process where the first transaction (from the old/current address) registers the new address (i.e. grants ownership) and the second transaction (from the new address) replaces the old address with the new one (i.e. claims ownership). This gives an opportunity to recover from incorrect addresses mistakenly used in the first step. If not, contract functionality might become inaccessible. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1488) and [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2369))

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

316:     function setLimit(uint256 _irrevocableLimit, uint256 _revocableLimit) external {

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

153:     function setTokensDistributionSpeed(address[] calldata tokens_, uint256[] calldata distributionSpeeds_) external {

177:     function setPrimeToken(address prime_) external onlyOwner {

310:     function _setTokenDistributionSpeed(address token_, uint256 distributionSpeed_) internal {

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-3] The critical parameters in initialize(...) are not set safely

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

130:     function initialize(

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

90:     function initialize(

```

### [L-4] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY

#### Description:

`initialize()` function can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the `__Ownable_init()` function. Also, there is no 0 address check in the address arguments of the initialize() function, which must be defined.

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

94:     ) external initializer {

```

### [L-5] MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS

#### Description:

Most contracts use the delegateCall proxy pattern and hence their implementations require the use of `initialize()` functions instead of constructors. This requires derived contracts to call the corresponding init functions of their inherited base contracts. This is done in most places except a few.

Impact: The inherited base classes do not get initialized which may lead to undefined behavior.

Missing call to `__ReentrancyGuard_init`

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

130:     function initialize(

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

90:     function initialize(

```

#### Recommended Mitigation Steps:

Add missing calls to init functions of inherited contracts.

### [L-6] Unify return criteria

#### Description:

In contracts, sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

add `{return x}` if you want to return the updated value or else remove `returns(uint)` from the `function(){}` if no value you wanted to return

```solidity
File: contracts/Tokens/Prime/Prime.sol

174:     function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {

433:     function claimInterest(address vToken) external whenNotPaused returns (uint256) {

443:     function claimInterest(address vToken, address user) external whenNotPaused returns (uint256) {

469:     function getAllMarkets() external view returns (address[] memory) {

478:     function claimTimeRemaining(address user) external view returns (uint256) {

496:     function calculateAPR(address market, address user) external view returns (uint256 supplyAPR, uint256 borrowAPR) {

533:     ) external view returns (uint256 supplyAPR, uint256 borrowAPR) {

597:     function getInterestAccrued(address vToken, address user) public returns (uint256) {

647:     function _calculateScore(address market, address user) internal returns (uint256) {

672:     function _claimInterest(address vToken, address user) internal returns (uint256) {

840:     function _xvsBalanceOfUser(address user) internal view returns (uint256) {

854:     function _xvsBalanceForScore(uint256 xvs) internal view returns (uint256) {

877:     ) internal view returns (uint256, uint256, uint256) {

904:     function isEligible(uint256 amount) internal view returns (bool) {

918:     function _interestAccrued(address vToken, address user) internal view returns (uint256) {

930:     function _getUnderlying(address vToken) internal view returns (address) {

947:     function _incomePerBlock(address vToken) internal view returns (uint256) {

957:     function _distributionPercentage() internal view returns (uint256) {

970:     function _incomeDistributionYearly(address vToken) internal view returns (uint256 amount) {

1001:     ) internal view returns (uint256 supplyAPR, uint256 borrowAPR) {

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

232:     function getEffectiveDistributionSpeed(address token_) external view returns (uint256) {

276:     function getBlockNumber() public view virtual returns (uint256) {

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath.sol

22:     function toFixed(uint256 n, uint256 d) internal pure returns (int256) {

34:     function uintDiv(uint256 u, int256 f) internal pure returns (uint256) {

46:     function uintMul(uint256 u, int256 f) internal pure returns (uint256) {

53:     function ln(int256 x) internal pure returns (int256 r) {

58:     function exp(int256 x) internal pure returns (int256 r) {

```

```solidity
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

51:     function ln(int256 x) internal pure returns (int256 r) {

140:     function exp(int256 x) internal pure returns (int256 r) {

```

```solidity
File: contracts/Tokens/Prime/libs/Scores.sol

27:     ) internal pure returns (uint256) {

```

### [L-7] TIMESTAMP DEPENDENCE

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

Reference: https://swcregistry.io/docs/SWC-116

Reference: (https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/timestamp-dependence.md)

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

378:             stakedAt[user] = block.timestamp;

399:         if (block.timestamp - stakedAt[msg.sender] < STAKING_PERIOD) revert WaitMoreTime();

481:         uint256 totalTimeStaked = block.timestamp - stakedAt[user];

```

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

277:         return block.number;

```

### [L-8] USE `_SAFEMINT` INSTEAD OF `_MINT`

#### Description:

According to openzepplin’s ERC721, the use of `_mint` is discouraged, use `safeMint` whenever possible.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-mint-address-uint256-

#### **Proof Of Concept**

```solidity
File: contracts/Tokens/Prime/Prime.sol

340:                     _mint(true, users[i]);

350:                 _mint(false, users[i]);

403:         _mint(false, msg.sender);

704:     function _mint(bool isIrrevocable, address user) internal {

```

#### Recommended Mitigation Steps:

Use `_safeMint` whenever possible instead of `_mint`
