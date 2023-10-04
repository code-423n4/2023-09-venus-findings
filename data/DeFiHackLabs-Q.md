## L-01. **maxLoopsLimit could end up being disadvantageous to protocol**

### Description:

The addMarket function can only be executed with admin permissions. Using _ensureMaxLoops to limit the maximum number of markets doesn't make much sense, especially since `prime.sol` doesn't have a setMaxLoopsLimit. If _ensureMaxLoops is set to 10, it means there can only ever be 10 markets.

### Instances:

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L109

```solidity
constructor(address _wbnb, address _vbnb, uint256 _blocksPerYear) {
        if (_wbnb == address(0)) revert InvalidAddress();
        if (_vbnb == address(0)) revert InvalidAddress();
        if (_blocksPerYear == 0) revert InvalidBlocksPerYear();
        WBNB = _wbnb;
        VBNB = _vbnb;
        BLOCKS_PER_YEAR = _blocksPerYear; 
```

Same issue reported last time : https://github.com/code-423n4/2023-05-venus-findings/issues/178

**Recommnendation:**

Add setMaxLoopsLimit function

```
    function setMaxLoopsLimit(uint256 limit) external onlyOwner {
        _setMaxLoopsLimit(limit);
    }
```

---

## L-02.  blocksPerYear without validation

### Description:

Venus Protocol is deployed on the BNB chain, which has a block-time of 15 seconds. Previously, the **`blocksPerYear`** constant was set without validation. To accurately reflect the number of blocks per year on the BNB chain, the **`blocksPerYear`** constant should be fixed. The correct value is 1,051,200.

### Instances:

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L109

```solidity
constructor(address _wbnb, address _vbnb, uint256 _blocksPerYear) {
        if (_wbnb == address(0)) revert InvalidAddress();
        if (_vbnb == address(0)) revert InvalidAddress();
        if (_blocksPerYear == 0) revert InvalidBlocksPerYear();
        WBNB = _wbnb;
        VBNB = _vbnb;
        BLOCKS_PER_YEAR = _blocksPerYear;

        // Note that the contract is upgradeable. Use initialize() or reinitializers
        // to set the state variables.
        _disableInitializers();
    }
```

**Recommnendation:**

Validate the correct `blocksPerYear` is 10512000

---

## L-03. Solidity outdated version

### Description:

All contracts are using 0.8.13. Consider updating to the latest version 0.8.21 to ensure the compiler contains the latest security fixes.

Storage Write Removal Bug On Conditional Early Termination
The bug was introduced in version 0.8.13 and Solidity version 0.8.17, released on September 08, 2022, provides a fix. The bug is significantly easier to trigger with optimized via-IR code generation, but can theoretically also occur in optimized legacy code generation.

REF: https://soliditylang.org/blog/2022/09/08/storage-write-removal-before-conditional-termination/

### Instances:

```solidity
// SPDX-License-Identifier: BSD-3-Clause
pragma solidity 0.8.13;
```

**Recommnendation:**

updating to the latest version 0.8.21 

---

## L-04. Openzeppelin outdated version

### Description:

Currently use OZ 4.8.3

This version has 6 medium issues
CVE-2023-40014
CVE-2023-34459
CVE-2023-34234
CVE-2023-40014
CVE-2023-34459
CVE-2023-34234

REF: https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/4.8.3

### Instances:

```solidity
@openzeppelin/contracts": "^4.8.3",
```

**Recommnendation:**

Upgrade to version 4.9.3 or higher.

## L-05.  Release funds without check pause status

### Description:

There is no implementation for fund transfers while paused in **`protocolShareReserve.sol`**. However, **`PrimeLiquidityProvider`** has a pause/unpause modifier. Yet, I don't see any checks for pause status in **`_claimInterest()`**. If **`FundsTransferIsPaused`** is set to paused, **`claimInterest`** will always revert.

### Instances:

https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L685C4-L685C4

```jsx
function _claimInterest(address vToken, address user) internal returns (uint256) {
        uint256 amount = getInterestAccrued(vToken, user);
        amount += interests[vToken][user].accrued;

        interests[vToken][user].rewardIndex = markets[vToken].rewardIndex;
        interests[vToken][user].accrued = 0;

        address underlying = _getUnderlying(vToken);
        IERC20Upgradeable asset = IERC20Upgradeable(underlying);

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

 
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192

```jsx
function releaseFunds(address token_) external {
        if (msg.sender != prime) revert InvalidCaller();
        if (paused()) {
            revert FundsTransferIsPaused();
        }
```

**Recommnendation:**

Check the pause status before performing an action.

## L-06.  initialize without contract integrity check

In initialize and constructor only check zero address, It is recommended to add a "checkContract" function to verify the integrity of the contract code during the deployment process. This can help prevent attacks that aim to exploit vulnerabilities in the contract's code or its dependencies. By including an integrity check, the contract can ensure that it is running as intended and that it has not been tampered with or modified in any way.

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L130

```jsx
constructor(address _wbnb, address _vbnb, uint256 _blocksPerYear) {
        if (_wbnb == address(0)) revert InvalidAddress();
        if (_vbnb == address(0)) revert InvalidAddress();
        if (_blocksPerYear == 0) revert InvalidBlocksPerYear();
        WBNB = _wbnb;
        VBNB = _vbnb;
        BLOCKS_PER_YEAR = _blocksPerYear;

        // Note that the contract is upgradeable. Use initialize() or reinitializers
        // to set the state variables.
        _disableInitializers();
    }
function initialize(
        address _xvsVault,
        address _xvsVaultRewardToken,
        uint256 _xvsVaultPoolId,
        uint128 _alphaNumerator,
        uint128 _alphaDenominator,
        address _accessControlManager,
        address _protocolShareReserve,
        address _primeLiquidityProvider,
        address _comptroller,
        address _oracle,
        uint256 _loopsLimit
    ) external virtual initializer {
        if (_xvsVault == address(0)) revert InvalidAddress();
        if (_xvsVaultRewardToken == address(0)) revert InvalidAddress();
        if (_protocolShareReserve == address(0)) revert InvalidAddress();
        if (_comptroller == address(0)) revert InvalidAddress();
        if (_oracle == address(0)) revert InvalidAddress();
        if (_primeLiquidityProvider == address(0)) revert InvalidAddress();
        _checkAlphaArguments(_alphaNumerator, _alphaDenominator);

        alphaNumerator = _alphaNumerator;
        alphaDenominator = _alphaDenominator;
        xvsVaultRewardToken = _xvsVaultRewardToken;
        xvsVaultPoolId = _xvsVaultPoolId;
        xvsVault = _xvsVault;
        nextScoreUpdateRoundId = 0;
        protocolShareReserve = _protocolShareReserve;
        primeLiquidityProvider = _primeLiquidityProvider;
        comptroller = _comptroller;
        oracle = ResilientOracleInterface(_oracle);

        __AccessControlled_init(_accessControlManager);
        __Pausable_init();
        _setMaxLoopsLimit(_loopsLimit);

        _pause();
    }
```

https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L90

```jsx
function initialize(
        address accessControlManager_,
        address[] calldata tokens_,
        uint256[] calldata distributionSpeeds_
    ) external initializer {
        __AccessControlled_init(accessControlManager_);
        __Pausable_init();

        uint256 numTokens = tokens_.length;
        if (numTokens != distributionSpeeds_.length) {
            revert InvalidArguments();
        }

        for (uint256 i; i < numTokens; ) {
            _initializeToken(tokens_[i]);
            _setTokenDistributionSpeed(tokens_[i], distributionSpeeds_[i]);

            unchecked {
                ++i;
            }
        }
    }
```

**Recommnendation:**

It is recommended to add a "checkContract" function.

Example:

```jsx
It is recommended to add a "checkContract" function.Example:
```

---

## L-07. APR Calculation for a user will fail when Oracle is paused.

**Description:**

In `Prime.sol`, the `calculateAPR/estimateAPR` function relies on fetching prices from the `ResilientOracle` to perform calculations. While this approach is generally effective, it presents a potential issue related to the `ResilientOracle`'s pausing or unavailability. If the oracle used in `calculateAPR/estimateAPR` is paused or experiences downtime, the function may not execute as expected, leading to inaccurate or incomplete results. This poses a risk to the contract's reliability and functionality.

**Recommendation:**

The `ResilientOracle` itself relies on multiple sources to fetch prices and has a fallback mechanism built into it. The ability to pause or unpause the oracle is under the control of the protocol's owners. Users must trust the protocol and hope that the protocol owners act responsibly to ensure the system's integrity. It's essential for protocol owners to maintain transparency and keep users informed about any actions related to the oracle to maintain user trust.

---

## L-08.  The protocol owner can `burn` the Prime SoulBound Token of any user

**Description:**

Prime Protocol includes a feature where users are rewarded with Prime SoulBound tokens when they stake 1000 XVS tokens for 90 days. While this incentive system is designed to encourage long-term participation, there is a potential centralization risk associated with the `burn` function.

The `burn` function allows the protocol owner to burn Prime SoulBound tokens, even if the user acquired them by fulfilling the staking requirement. This centralization of power in the hands of the protocol owner poses a risk to the fairness and trustworthiness of the protocol.

The test case titled *`manually burn irrevocable token`* demonstrates the issue.

**Recommendation:**

Users must trust the protocol and hope that the protocol owners act responsibly to ensure the system's integrity. It's essential for protocol owners to maintain transparency and keep users informed about any actions related to the oracle to maintain user trust.

"manually burn irrevocable token"

---