# QA Report for Venus Prime

## Overview

During the audit, 4 Low issues were found.

### Low-Risk Issues

| # | Issue | Instances |
| --- | --- | --- |
| [L-01](#l-01-Missing-Minimum-Stake-Check-in-primeclaim) | Missing Minimum Stake Check in `Prime::claim()` | 1 |
| [L-02](#l-02-Unavailability-of-Fund-Retrieval-in-Primesol) | Unavailability of Fund Retrieval in `Prime.sol` | 1 |
| [L-03](#l-03-pendingScoreUpdates-is-not-decremented-in-_updateScore) | `pendingScoreUpdates` is not decremented in `_updateScore` | 1 |
| [L-04](#l-04-Missing-setters-for-important-variables) | Missing setters for important variables | 6 |

**Total** 9 instances over 4 issues

------

### [L-01] Missing Minimum Stake Check in Prime::claim()

**Description**

- Within `Prime::claim()`, the verification for the minimum staked amount is absent.
- This problem also contributes to the scenario where a staker with an amount below the minimum staked requirement can claim the revocable token.
- The presence of this check would have effectively prevented such a situation from occurring.

**Recommendation**

- Add a check for minimum staked amount:

```solidity
function claim() external {
	if (stakedAt[msg.sender] == 0) revert IneligibleToClaim();
	//Add the following check
	uint256 totalStaked = _xvsBalanceOfUser(user);
	if(!isEligible(totalStaked)) revert NotEnoughStake();

	if (block.timestamp - stakedAt[msg.sender] < STAKING_PERIOD) revert WaitMoreTime();

	stakedAt[msg.sender] = 0;

  _mint(false, msg.sender);
  _initializeMarkets(msg.sender);
}
```

### [L-02] Unavailability of Fund Retrieval in `Prime.sol`

**Description**

- The Prime contract can access funds from ProtocolShareReserve and PrimeLiquidityProvider as needed.
- However, in a scenario where an upgrade is required and the Prime contract holds excess tokens due to calls, such as `IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets)` and `IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset))`, there is no mechanism to retrieve these surplus funds.

**Recommendation**

- Incorporate a function akin to [`PrimeLiquidityProvider::sweepToken()`](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216) within `Prime.sol`.
- **Caution**: Ensure that this function is utilized only when there are no further rewards or interests to claim, as its use under other circumstances WILL disrupt accounting.

### [L-03] `pendingScoreUpdates` is not decremented in `_updateScore`

**Description**

- When an update to the multiplier or alpha value triggers the initiation of a new score update round using the `Prime::_startScoreUpdateRound()` function, the variables `pendingScoreUpdates` and `totalScoreUpdatesRequired` are employed for community tracking.

```solidity
function _startScoreUpdateRound() internal {
		nextScoreUpdateRoundId++;
		totalScoreUpdatesRequired = totalIrrevocable + totalRevocable;
		pendingScoreUpdates = totalScoreUpdatesRequired;
}
```

- However, a discrepancy arises as the `pendingScoreUpdates` variable remains unaltered when users manually update their scores, resulting in inaccurate tracking.

**Recommendation**

- Amend the following function in `Prime.sol`:

```solidity
 function _updateScore(address user, address market) internal {
		if (!markets[market].exists || !tokens[user].exists) {
				return;
		}

		uint256 score = _calculateScore(market, user);
		markets[market].sumOfMembersScore = markets[market].sumOfMembersScore - interests[market][user].score + score;
		interests[market][user].score = score;

		//Incorporate the following to ensure precise tracking
		if (pendingScoreUpdates > 0 && !isScoreUpdated[nextScoreUpdateRoundId][user]) {
				pendingScoreUpdates--;
		}
}
```

### [L-04] Missing Setters For Important Variables

**Description**

- Missing setters for important variables in `Prime.sol`:
    - `xvsVault`
    - `xvsVaultRewardToken`
    - `protocolShareReserve`
    - `primeLiquidityProvider`
    - `comptroller`
    - `oracle`

**Recommendation**

- Add the following functions in `Prime.sol`

```solidity
// Setter for xvsVault with access control
function setXVSVault(address _xvsVault) external {
    _checkAccessAllowed("setXVSVault(address)");
    xvsVault = _xvsVault;
}

// Setter for xvsVaultRewardToken with access control
function setXVSVaultRewardToken(address _xvsVaultRewardToken) external {
    _checkAccessAllowed("setXVSVaultRewardToken(address)");
    xvsVaultRewardToken = _xvsVaultRewardToken;
}

// Setter for protocolShareReserve with access control
function setProtocolShareReserve(address _protocolShareReserve) external {
    _checkAccessAllowed("setProtocolShareReserve(address)");
    protocolShareReserve = _protocolShareReserve;
}

// Setter for primeLiquidityProvider with access control
function setPrimeLiquidityProvider(address _primeLiquidityProvider) external {
    _checkAccessAllowed("setPrimeLiquidityProvider(address)");
    primeLiquidityProvider = _primeLiquidityProvider;
}

// Setter for comptroller with access control
function setComptroller(address _comptroller) external {
    _checkAccessAllowed("setComptroller(address)");
    comptroller = _comptroller;
}

// Setter for oracle with access control
function setOracle(address _oracle) external {
    _checkAccessAllowed("setOracle(address)");
    oracle = _oracle;
}
```