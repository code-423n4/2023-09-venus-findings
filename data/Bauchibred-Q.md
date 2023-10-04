# QA Report

## **Table of Contents**

|       | Issue                                                                                                                 |
| ----- | --------------------------------------------------------------------------------------------------------------------- |
| QA-01 | `_claimInterest()` in Prime.sol should include a slippage mechanism or allow users to claim their interest partially  |
| QA-02 | Improvements required for `getEffectiveDistributionSpeed()` to handle cases where `accrued` is greater than `balance` |
| QA-03 | Discrepancies noted in parameter validations among sister functions                                                   |
| QA-04 | The `lastAccruedBlock` naming needs to be refactored                                                                  |
| QA-05 | Additional sanity checks are recommended for the `calculateScore()` function                                          |
| QA-06 | Suggestions for better handling of unavailable markets in the `getPendingInterests()` function                        |
| QA-07 | Implementation gaps observed due to missing zero and isContract checks in critical functions                          |
| QA-08 | Essential to protect against division by zero in the `uintDiv()` function from `FixedMath.sol`                        |
| QA-09 | Fix typo in docs                                                                                                      |

## QA-01 Prime.sol's `_claimInterest()` should apply a slippage or allow users to partially claim their interest

### Impact

_Medium to low_ The `_claimInterest` function hardcodes a 0% percent slippage and does not perform a final check after attempting to release funds from various sources. If the contract's balance is still insufficient after these fund releases, it could lead to a revert on the `safeTransfer` call, preventing users from claiming their interest.

### Proof of Concept

Take a look at the `_claimInterest()` function:

```solidity
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
      //@audit-issue M no extra check to see if the balance is still not enough, in the case where it isn't then the attempt to safetransfer the tokens  would always revert so at least available balance could be sent to user and then later on extras get sent, no?
      unreleasedPLPIncome[underlying] = 0;
    }
  }

  asset.safeTransfer(user, amount);

  emit InterestClaimed(user, vToken, amount);

  return amount;
}
```

In the above function, the contract tries to ensure it has enough balance to pay out the user's interest in the following way:

1. If the required `amount` is greater than the contract's balance, it releases funds from `IProtocolShareReserve`.
2. If the amount is still greater, it attempts to release funds from `IPrimeLiquidityProvider`.

The potential issue arises after this. There is no final check to ensure that the `amount` is less than or equal to the contract's balance before executing `safeTransfer`, do note that a slippage of 0% has been applied.

If both releases (from `IProtocolShareReserve` and `IPrimeLiquidityProvider`) do not provide sufficient tokens, the `safeTransfer` will revert, and the user will be unable to claim their interest.

Depending on the situation this might not only lead to user frustration but also cause trust issues with the platform and would lead to user funds to be stuck s

### Recommended Mitigation Steps

To address this issue, the following changes are suggested:

- Add a final balance check.
- If the balance is still insufficient, send whatever is available to the user.
- Store the deficit somewhere and inform the user that there's more to claim once the contract has sufficient funds.

A diff for the recommended changes might look like:

```diff
function _claimInterest(address vToken, address user) internal returns (uint256) {
    ...
    if (amount > asset.balanceOf(address(this))) {
        address[] memory assets = new address[](1);
        assets[0] = address(asset);
        IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
        if (amount > asset.balanceOf(address(this))) {
            IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
            unreleasedPLPIncome[underlying] = 0;
        }
    }

+   uint256 availableBalance = asset.balanceOf(address(this));
+   uint256 transferAmount = (amount <= availableBalance) ? amount : availableBalance;

-   asset.safeTransfer(user, amount);
+   asset.safeTransfer(user, transferAmount);

    emit InterestClaimed(user, vToken, transferAmount);

    return transferAmount;
}
```

This ensures that a user always gets whatever is available, if the third suggestion is going to be implemented then do not forget to store the differences in a variable

> NB: Submitting as QA, due to the chances of this happening being relatively low, but leaving the final decision to judge to upgrade to Medium if they see fit.

## QA-02 `getEffectiveDistributionSpeed()` should prevent reversion and return `0` if `accrued` is ever more than `balance`

### Impact

Low, since this is just how to better implement `getEffectiveDistributionSpeed()` to cover more valid _edge cases_.

### Proof of Concept

Take a look at `getEffectiveDistributionSpeed()`:

```solidity
function getEffectiveDistributionSpeed(address token_) external view returns (uint256) {
  uint256 distributionSpeed = tokenDistributionSpeeds[token_];
  uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
  uint256 accrued = tokenAmountAccrued[token_];

  if (balance - accrued > 0) {
    return distributionSpeed;
  }

  return 0;
}
```

As seen, the above function is used to get the rewards per block for a specific token; the issue is that an assumption is made that `balance >= accrued` is always true, which is not the case.

To support my argument above, note that the `PrimeLiquidityProvider.sol` also employs functions like `sweepTokens()` and `releaseFunds`. Whereas the latter makes an update to the accrual during its execution (i.e., setting the `tokenAmountAccrued[token_]` to `0`), the former does not. This can be seen below:

```solidity
function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
  uint256 balance = token_.balanceOf(address(this));
  if (amount_ > balance) {
    revert InsufficientBalance(amount_, balance);
  }

  emit SweepToken(address(token_), to_, amount_);

  token_.safeTransfer(to_, amount_);
}

function releaseFunds(address token_) external {
  if (msg.sender != prime) revert InvalidCaller();
  if (paused()) {
    revert FundsTransferIsPaused();
  }

  accrueTokens(token_);
  uint256 accruedAmount = tokenAmountAccrued[token_];
  tokenAmountAccrued[token_] = 0;

  emit TokenTransferredToPrime(token_, accruedAmount);

  IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);
}
```

Now, in a case where a portion of a token has been swept and the accrual is now more than the balance of said token, the `getEffectiveDistributionSpeed()` function reverts due to an underflow from the check here:

```solidity
  uint256 distributionSpeed = tokenDistributionSpeeds[token_];
  uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
  uint256 accrued = tokenAmountAccrued[token_];

  if (balance - accrued > 0)
```

### Recommended Mitigation Steps

Make an addition to cover up the edge case in the `getEffectiveDistributionSpeed()` function, i.e., if accrued is already more than the balance then the distribution speed should be the same as if the difference is 0.

An approach to the recommended fix can be seen from the diff below:

```diff
    function getEffectiveDistributionSpeed(address token_) external view returns (uint256) {
        uint256 distributionSpeed = tokenDistributionSpeeds[token_];
        uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
        uint256 accrued = tokenAmountAccrued[token_];
-        if (balance - accrued > 0) {
+        if (balance > accrued) {
            return distributionSpeed;
        }

        return 0;
    }
```

---

## QA-03 Inconsistency in parameter validations between sister functions

### Impact

_Low, info._

The inconsistency in validation checks between similar functions can lead to unexpected behavior.

### Proof of Concept

Take a look at `updateAlpha()`:

```solidity
function updateAlpha(uint128 _alphaNumerator, uint128 _alphaDenominator) external {
  _checkAccessAllowed("updateAlpha(uint128,uint128)");
  _checkAlphaArguments(_alphaNumerator, _alphaDenominator);

  emit AlphaUpdated(alphaNumerator, alphaDenominator, _alphaNumerator, _alphaDenominator);

  alphaNumerator = _alphaNumerator;
  alphaDenominator = _alphaDenominator;

  for (uint256 i = 0; i < allMarkets.length; ) {
    accrueInterest(allMarkets[i]);

    unchecked {
      i++;
    }
  }

  _startScoreUpdateRound();
}

function updateMultipliers(address market, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
  _checkAccessAllowed("updateMultipliers(address,uint256,uint256)");
  if (!markets[market].exists) revert MarketNotSupported();

  accrueInterest(market);

  emit MultiplierUpdated(
    market,
    markets[market].supplyMultiplier,
    markets[market].borrowMultiplier,
    supplyMultiplier,
    borrowMultiplier
  );
  markets[market].supplyMultiplier = supplyMultiplier;
  markets[market].borrowMultiplier = borrowMultiplier;

  _startScoreUpdateRound();
}
```

In the `updateAlpha` function, there is a check `_checkAlphaArguments(_alphaNumerator, _alphaDenominator)` which ensures the correctness of the arguments provided. This establishes a certain standard of validation for updating alpha values.

```solidity
function _checkAlphaArguments(uint128 _alphaNumerator, uint128 _alphaDenominator) internal {
  if (_alphaDenominator == 0 || _alphaNumerator > _alphaDenominator) {
    revert InvalidAlphaArguments();
  }
}
```

However, in the `updateMultipliers` function, there is no equivalent validation function like `_checkAlphaArguments`. This discrepancy implies that the two functions, which are logically do not maintain a consistent standard of input validation.

### Recommended Mitigation Steps

Consider introducing a validation function (e.g., `_checkMultipliersArguments`) that will validate the parameters `supplyMultiplier` and `borrowMultiplier`.

- Incorporate this validation function into the `updateMultipliers` function, similar to how `_checkAlphaArguments` is used in the `updateAlpha` function.
- Make sure that the validation function checks for logical constraints, edge cases, or any other relevant criteria for these parameters.

A diff for the recommended changes might appear as:

```diff
+ function _checkMultipliersArguments(uint256 supplyMultiplier, uint256 borrowMultiplier) internal {
+     // Implement the validation logic here
+     ...
+ }

function updateMultipliers(address market, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
    ...
+   _checkMultipliersArguments(supplyMultiplier, borrowMultiplier);
    ...
    if (!markets[market].exists) revert MarketNotSupported();
    ...
    emit MultiplierUpdated(
        ...
    );
    markets[market].supplyMultiplier = supplyMultiplier;
    markets[market].borrowMultiplier = borrowMultiplier;

    _startScoreUpdateRound();
}
```

## QA-04 The `lastAccruedBlock` naming needs to be refactored

### Impact

Low, confusing comment/docs since the `lastAccruedBlock` is either wrongly implemented or wrongly named.

### Proof of Concept

Take a look at [PrimeLiquidityProvider.sol#L22-L24](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L22-L24):

```soldidity
    /// @notice The rate at which token is distributed to the Prime contract
    mapping(address => uint256) public lastAccruedBlock;
```

As seen, `lastAccruedBlock` currently claims to be the rate at which a token is distributed to the prime contract, but this is wrong since this mapping actuallly holds value for the last block an accrual was made for an address

### Recommended Mitigation Steps

Fix the comments arounf the `lastAccruedBlock` mapping or change the name itself

## QA-05 More sanity checks should be applied to `calculateScore()`

### Impact

Low, info since the chances of passing a valuse above type(int256).max is relatively slim

### Proof of Concept

Take a look at `calculateScore()`:

```solidity
function calculateScore(
  uint256 xvs,
  uint256 capital,
  uint256 alphaNumerator,
  uint256 alphaDenominator
) internal pure returns (uint256) {
  // Score function is:
  // xvs^𝝰 * capital^(1-𝝰)
  //    = capital * capital^(-𝝰) * xvs^𝝰
  //    = capital * (xvs / capital)^𝝰
  //    = capital * (e ^ (ln(xvs / capital))) ^ 𝝰
  //    = capital * e ^ (𝝰 * ln(xvs / capital))     (1)
  // or
  //    = capital / ( 1 / e ^ (𝝰 * ln(xvs / capital)))
  //    = capital / (e ^ (𝝰 * ln(xvs / capital)) ^ -1)
  //    = capital / e ^ (𝝰 * -1 * ln(xvs / capital))
  //    = capital / e ^ (𝝰 * ln(capital / xvs))     (2)
  //
  // To avoid overflows, use (1) when xvs < capital and
  // use (2) when capital < xvs

  // If any side is 0, exit early
  if (xvs == 0 || capital == 0) return 0;

  // If both sides are equal, we have:
  // xvs^𝝰 * capital^(1-𝝰)
  //    = xvs^𝝰 * xvs^(1-𝝰)
  //    = xvs^(𝝰 + 1 - 𝝰)     = xvs
  if (xvs == capital) return xvs;

  bool lessxvsThanCapital = xvs < capital;

  // (xvs / capital) or (capital / xvs), always in range (0, 1)
  int256 ratio = lessxvsThanCapital ? FixedMath.toFixed(xvs, capital) : FixedMath.toFixed(capital, xvs);

  // e ^ ( ln(ratio) * 𝝰 )
  int256 exponentiation = FixedMath.exp(
    (FixedMath.ln(ratio) * alphaNumerator.toInt256()) / alphaDenominator.toInt256()
  );

  if (lessxvsThanCapital) {
    return FixedMath.uintMul(capital, exponentiation);
  }

  // capital / e ^ (𝝰 * ln(capital / xvs))
  return FixedMath.uintDiv(capital, exponentiation);
}
```

Do note that the function is used to calculate a membership score given some amount of `xvs` and `capital`, issue is that both alpha numerators and denoimantors are passed in as `uint256` values and then while getting the `exponentiation` value these values are converted to `int256` which would cause a revert.

### Recommended Mitigation Steps

Better code structure could be applied, from the get go the values passed in for numerators and denominators could be checked to ensure that they are not above the max value of int256

## QA-06 The `getPendingInterests()` should be better implemented to handle cases where a market is unavailable.

### Impact

Low.

If one market goes down or becomes unresponsive, users won't be able to retrieve their pending interests for any of the subsequent markets. This can lead to misinformation and might affect user decisions.

### Proof of Concept

Take a look at`getPendingInterests()`:

```solidity
function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
  address[] storage _allMarkets = allMarkets;
  PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);
  //@audit-ok this function currently does not take into account that one of the markets could go down in that case a break in this loop could occur, check perennial sherlock

  for (uint256 i = 0; i < _allMarkets.length; ) {
    address market = _allMarkets[i];
    uint256 interestAccrued = getInterestAccrued(market, user);
    uint256 accrued = interests[market][user].accrued;

    pendingInterests[i] = PendingInterest({ market: IVToken(market).underlying(), amount: interestAccrued + accrued });

    unchecked {
      i++;
    }
  }

  return pendingInterests;
}
```

As seen, there's a loop that goes through all available markets to fetch the boosted pending interests for a user. However, if any of these calls (like `getInterestAccrued`) fail due to market unavailability or any other issue, the loop will be interrupted.

### Recommended Mitigation Steps

To ensure the function's resilience and provide accurate information even if some markets are down, wrap the calls inside the loop in a `try/catch` block. If a call fails, log an event indicating the market that caused the failure:

```solidity
event MarketCallFailed(address indexed market);

for (uint256 i = 0; i < _allMarkets.length; ) {
    address market = _allMarkets[i];

    try {
        uint256 interestAccrued = getInterestAccrued(market, user);
        uint256 accrued = interests[market][user].accrued;

        pendingInterests[i] = PendingInterest({
            market: IVToken(market).underlying(),
            amount: interestAccrued + accrued
        });
    } catch {
        emit MarketCallFailed(market);
    }

    unchecked {
        i++;
    }
}
```

By incorporating this change, even if a market fails, the loop won't break, and the function will continue to retrieve the pending interests for the remaining markets. The emitted event will notify off-chain systems or users about the problematic market.

## QA-07 Missing zero/isContract checks in important functions

### Impact

Low since this requires a bit of an admin error, but some functions which could be more secure using the `_ensureZeroAddress()` currently do not implement this and could lead to issues.

### Proof of Concept

Using the `PrimeLiquidityProvider.sol` in scope as acase study.
Below is the implementation of `_ensureZeroAddress()`

```solidity
function _ensureZeroAddress(address address_) internal pure {
  if (address_ == address(0)) {
    revert InvalidArguments();
  }
}
```

As seen the above is used within protocol as a modifier/function to revert whenever addresses are being passed, this can be seen to be implemented in the `setPrime()` function and others, but that's not always the case and in some instances addresses are not ensured to not be zero.

Additionally, as a security measure an `isContract()` function could be added and used to check for instances where the provided addresses must be valid contracts with code.

### Recommended Mitigation Steps

Make use of `_ensureZeroAddress()` in all instances where addresses could be passed.

If the `isContract()` is going to be implemented then the functionality to be added could look something like this:

```solidity
function isContract(address addr) internal view returns (bool) {
  uint256 size;
  assembly {
    size := extcodesize(addr)
  }
  return size > 0;
}
```

And then, this could be applied to instances where addreses must have byte code.

## QA-08 The `uintDiv()` function from `FixedMath.sol` should protect against division by `0`

### Impact

Low, just info on how to prevent panic errors cause by `division by zero`

### Proof of Concept

Take a look at `uintDiv()`:

```solidity
function uintDiv(uint256 u, int256 f) internal pure returns (uint256) {
  if (f < 0) revert InvalidFixedPoint(); //@audit
  // multiply `u` by FIXED_1 to cancel out the built-in FIXED_1 in f
  return uint256((u.toInt256() * FixedMath0x.FIXED_1) / f);
}
```

As seen, this function is used to divide an unsigned `int` in this case `u` by a fixed number `f`, but currently the only check applied to the `uintDiv()` function in regards to the divisor `f` is:
`        if (f < 0) revert InvalidFixedPoint();`
The above means that a value of `0` would get accepted for `f` which would result in a panic error.

### Recommended Mitigation Steps

Make changes to the `uintDiv()` function

```diff
    function uintDiv(uint256 u, int256 f) internal pure returns (uint256) {
-        if (f < 0) revert InvalidFixedPoint();
+        if (f <= 0) revert InvalidFixedPoint();
        // multiply `u` by FIXED_1 to cancel out the built-in FIXED_1 in f
        return uint256((u.toInt256() * FixedMath0x.FIXED_1) / f);
    }
```

## QA-09 Fix typo in docs

### Impact

Low, info on how to have a better code documentation

### Proof of Concept

Take a look at the Prime token's [ReadMe](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/README.md?plain=1#L295-L296)

Line 296 states:

```
The OpenZeppelin Plausable contract is used. Only the `claimInterest` function is under control of this pause mechanism.
```

As seen an evident typo has been missed.

### Recommended Mitigation Steps

Change the line, from:

```
The OpenZeppelin Plausable contract is used. Only the `claimInterest` function is under control of this pause mechanism.
```

To:

```
The OpenZeppelin Pausable contract is used. Only the `claimInterest` function is under control of this pause mechanism.
```
