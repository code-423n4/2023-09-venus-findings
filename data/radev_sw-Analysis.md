# Analysis Report

---

## Here is my notes during the contest

---

## Notes for PrimeStorage contract

1. **Structs**:

   - **Token**: This structure appears to describe metadata for a token. The `exists` flag signifies if a token exists, and the `isIrrevocable` flag denotes whether the token can be revoked.

   - **Market**: Represents a market with configuration parameters such as multipliers for supply and borrowing, an index for rewards, a sum of member scores, and an existence flag.

   - **Interest**: Appears to be a record of accrued interest, a score, and an index related to rewards.

   - **PendingInterest**: A structure to keep track of interest that's pending on a particular market for a user.

2. **Constants**:

   - **EXP_SCALE**: Often represents a scaling factor for handling fixed-point arithmetic in contracts. `1e18` is typical for Ethereum contracts to represent decimals since Solidity does not have native decimal support.

   - **MINIMUM_STAKED_XVS**: Represents the minimum amount of XVS tokens that a user needs to stake to become a prime member.

   - **MAXIMUM_XVS_CAP**: The maximum amount of XVS tokens that are considered when calculating a user's score.

   - **STAKING_PERIOD**: The duration a user needs to stake to be eligible to claim a prime token, in seconds. The provided value corresponds to 90 days.

   - **MAXIMUM_BPS**: Represents the maximum Basis Points (bps). 10000 bps is equivalent to 100%.

3. **Mappings and Arrays**:

   - **tokens**: A mapping to get the metadata of a prime token.

   - **totalIrrevocable** and **totalRevocable**: Track the total number of irrevocable and revocable tokens minted, respectively.

   - **revocableLimit** and **irrevocableLimit**: Represent the maximum number of revocable and irrevocable tokens that can be minted, respectively.

   - **stakedAt**: Tracks when eligible users started staking for claiming prime tokens.

   - **markets**: A mapping of a token (probably representing a market) to its configuration.

   - **interests**: Keeps track of user interests per market.

   - **allMarkets**: A list of all markets that have received a boost.

   - **alphaNumerator** and **alphaDenominator**: Components of a fraction representing 'alpha'. This could be used for calculations like weighted averages or other formulas.

   - **xvsVault**, **xvsVaultRewardToken**, and **xvsVaultPoolId**: Details related to the XVS vault. This could be for managing interactions with a reward vault related to the XVS token.

   - **isScoreUpdated**: Checks if a user's score was updated in a particular round.

   - **nextScoreUpdateRoundId**, **totalScoreUpdatesRequired**, **pendingScoreUpdates**: Variables related to score updates. These help manage rounds of score updates and track pending updates.

   - **vTokenForAsset**: Maps an asset to its corresponding vToken (possibly representing a tokenized version or a wrapped token in the ecosystem).

   - **protocolShareReserve**, **comptroller**, **primeLiquidityProvider**: Addresses of important contracts or entities in the ecosystem.

   - **unreleasedPSRIncome** and **unreleasedPLPIncome**: Represent undistributed income from PSR (Protocol Share Reserve) and PLP (Prime Liquidity Provider) respectively, which has been allocated to prime token holders but not yet released.

   - **oracle**: The address of the Resilient Oracle contract, which is presumably used for price feed or other data requirements.

4. **Reserved Space**:

   - **\_\_gap**: This is a common pattern in upgradeable smart contracts. It reserves storage space to ensure that future versions of the contract can introduce new state variables without causing storage collisions.

---

---

## Notes for Prime contract

revocable tokens: if the user unstakes some of their XVS, and the new staked amount is below 1,000 XVS, then the token is burnt
irrevocable tokens: they are not burnt when the total amount of XVS staked is below 1,000 XVS

irrevocable tokens can be minted only via VIP (Venus Improvement Proposal), after a vote of the Venus community. Revocable tokens should be "easier" to get: as soon as the user satisfies the requirements (staking more than 1,000 XVS for 90 days), a revocable token can be claimed by the user. It doesn't need a VIP

the benefit of a irrevocable token is that it doesn't depend on the staked XVS. If you have a irrevocable Prime token, you can have only 500 XVS staked in the XVSVault, for example (this is from the technical point of view; from the business point of view, probably user with irrevocable Prime tokens already have a lot of XVS staked in the vault)

the criteria to receive an irrevocable Prime token are not specified yet, but they would be be harder to satisfy than the revocable tokens criteria

moreover, take into account that a revocable token can be "upgrade" to be irrevocable (via VIP), but an irrevocable token cannot be downgrade to be revocable

regarding rewards, both tokens are considered the same way, and the same inputs (XVS staked and borrowed and supplied amounts) will be taken into account to calculate the rewards

![Alt text](image.png)

---

The user deposits at least 1000 XVS in XVSVault.sol (out of scope) with the deposit function. Then waits 90 days and claims his Prime token with the claim function in Prime.sol. Once the use has a prime token he is basically receiving interest that can be collected at any time with claimInterest in Prime.sol. Understanding the logic behind the reward system is up to you.

---

What is the basic difference between score and interest

- Interest is the amount of rewards generated per 1 point of score (ex. 2 USDC) and score is the points each user has.
  Example:
  User1 score-10 && interest-2 USDC => reward of 20 USDC

---

`xvsUpdated*()` function and analyze all possible conditions. I'll present the findings in a tabular format for clarity.

To facilitate this, I'll define the possible states for each variable:

- `totalStaked` can be `> MINIMUM_STAKED_XVS` or `<= MINIMUM_STAKED_XVS`. For simplicity, let's assume `MINIMUM_STAKED_XVS` is `1000`.
- `isAccountEligible` can be `true` or `false`.
- `tokens[user].exists` can be `true` or `false`.
- `stakedAt[user]` can be `> 0` or `= 0`.

| totalStaked | isAccountEligible | tokens[user].exists | stakedAt[user] | Path Taken in `xvsUpdated`                                                         |
| ----------- | ----------------- | ------------------- | -------------- | ---------------------------------------------------------------------------------- |
| > 1000      | true              | true                | > 0            | \_accrueInterestAndUpdateScore                                                     |
| > 1000      | true              | true                | = 0            | \_accrueInterestAndUpdateScore                                                     |
| > 1000      | true              | false               | > 0            | No action taken                                                                    |
| > 1000      | true              | false               | = 0            | `stakedAt[user] = block.timestamp`                                                 |
| <= 1000     | false             | true                | > 0            | If `tokens[user].isIrrevocable`, then \_accrueInterestAndUpdateScore. Else, \_burn |
| <= 1000     | false             | true                | = 0            | If `tokens[user].isIrrevocable`, then \_accrueInterestAndUpdateScore. Else, \_burn |
| <= 1000     | false             | false               | > 0            | `stakedAt[user] = 0`                                                               |
| <= 1000     | false             | false               | = 0            | No action taken                                                                    |

---

---

---

## Documentation Notes

### Expected Impact and Launch

**Goal**: Increase users' staking of XVS tokens, grow the total value locked in the platform, and enhance market growth.

**How**:

- Reward users who stake larger amounts.
- Encourage users to keep their tokens staked for longer periods.
- Attract diverse users to participate.

### Rewards

- Uses the **Cobb-Douglas function**, which is inspired by the Goldfinch rewards mechanism.
- This formula decides how rewards are distributed among users.

**Main Factors in the Reward Calculation**:

- The amount of XVS a user stakes.
- The total lending and borrowing a user does on the platform.

_Note_:

1. Only a certain amount of staked XVS is considered when calculating rewards.
2. The amount a user can lend or borrow that is considered for rewards is based on the amount of XVS they've staked.

**Significance of α**:

- It's a value between 0 and 1 that determines the weightage of staked tokens versus borrowed/supplied tokens in reward calculations.
- Default value is 0.5, which gives equal importance to staking and borrowing/supplying. But this can change if needed.

### Implementation of Rewards in Solidity (Technical Part)

- A global `rewardIndex` keeps track of the rewards rate for each market.
- `sumOfMembersScore` is the total reward score of all users.
- Rewards for each user are calculated based on the difference between the global reward index and the user's own reward index.

### Income Collection and Distribution

- A portion of the protocol's income is reserved for Prime token holders.
- This reserved income is released to the `Prime` contract every 10 blocks, which then distributes it to users based on their scores.
- If the contract doesn’t have enough funds when a user tries to claim rewards, additional funds are released from the reserve in the same transaction.

### Update Cap Multipliers and Alpha

- The values that determine how much a user can borrow or lend (multipliers) and the weightage of staking versus borrowing/supplying (alpha) can be changed.
- When they're changed, a script updates the scores of all users to reflect these changes. This prevents large rewards from going to a small group of users.

### Calculate APR Associated with a Prime Market and User

- It provides a way to estimate the yearly return (APR) a user can expect based on their activity and the current rewards rate.
- This APR is based on the protocol's income, the percentage of that income sent to the Prime rewards program, and the user's activity (staking, borrowing, supplying).

**Conclusion**:
The Venus Prime protocol encourages users to stake their XVS tokens and engage in borrowing and lending activities on the platform. It has a well-defined rewards mechanism that benefits active and loyal participants. The protocol is transparent about how rewards are calculated and distributed, and provides tools for users to estimate their potential returns.


### Time spent:
30 hours