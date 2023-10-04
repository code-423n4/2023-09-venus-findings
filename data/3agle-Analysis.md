# Analysis of Venus Prime

## Index
- [Codebase Quality](#codebase-quality)
	- [Strengths](#strengths)
	- [Weaknesses](#weaknesses)
- [Mechanism Review](#mechanism-review)
	- [Architecture & Working](#architecture--working)
	- [Reward Mechanism](#reward-mechanism)
	- [Prime](#prime)
	- [Prime Liquidity Provider](#primeliquidityprovider)
- [Architecture Recommendations](#architecture-recommendations)
-  [Centralization Risks](#centralization-risks)
- [Systemic Risks](#systemic-risks)
- [Key Takeaways](#key-takeaways)

---

## Codebase quality

- The codebase exhibited remarkable quality. It has undergone multiple audits, ensuring the robustness of all major functionalities.

### **Strengths**

- Comprehensive unit tests
- Utilization of Natspec
- Adoption of governance for major changes, reducing centaralization risks.

### **Weaknesses**

- Documentation had room for improvement.

## Mechanism Review

### **Architecture & Working**

- Following are the main components of the Venus Prime system:
    1. **Prime.sol:**
        - *Description*: Prime.sol serves as the central contract responsible for distributing both revocable and irrevocable tokens to stakers.
        - *Functionality*: This contract plays a pivotal role in issuing Prime tokens to eligible stakers, enabling them to participate in the Venus Prime program and earn rewards.
    2. **ProtocolLiquidityProvider:**
        - *Description*: The ProtocolLiquidityProvider contract holds a reserve of funds that are designated for gradual allocation to the Prime contract over a predefined time period.
        - *Functionality*: This component ensures a consistent and planned provision of tokens to the Prime contract, guaranteeing a continuous supply of rewards for Prime token holders.
    3. **ProtocolShareReserve (OOS):**
        - *Description*: The ProtocolShareReserve acts as the repository for interest earnings generated across all prime markets within the Venus Protocol.
        - *Functionality*: It collects the interest earnings and acts as a centralized pool from which the Prime contract draws the necessary funds. These funds are then distributed as rewards to Prime token holders, contributing to the sustainability of the Venus Prime program.

![https://res.cloudinary.com/duqxx6dlj/image/upload/v1696427287/VenusPrime/Untitled_ezmccy.png](https://res.cloudinary.com/duqxx6dlj/image/upload/v1696427287/VenusPrime/Untitled_ezmccy.png)

### **Rewarding Mechanism**

![https://res.cloudinary.com/duqxx6dlj/image/upload/v1696427286/VenusPrime/Untitled_1_knefs7.png](https://res.cloudinary.com/duqxx6dlj/image/upload/v1696427286/VenusPrime/Untitled_1_knefs7.png)

1. Initialization (Block 0):
    - Variables are set to default values.
2. Income and Scoring (Blocks 1-10):
    - Over 10 blocks, 20 USDT income is distributed.
    - User scores total 50, allocating 0.4 USDT per score point.
    - `markets[vToken].rewardIndex` increases by +0.4.
3. Income and Scoring (Blocks 11-30):
    - Next 20 blocks distribute 10 USDT.
    - User scores remain constant.
    - `markets[vToken].rewardIndex` increases by +0.2.
4. User Claims a Prime Token (Block 35):
    - User claims a Prime token.
    - `markets[vToken].rewardIndex` updated via `accrueInterest`.
    - Over 5 blocks, 5 USDT income with no score change.
    - `markets[vToken].rewardIndex` increases by +0.1.
    - `interests[market][account].rewardIndex` set to `markets[market].rewardIndex`.
5. User Claims Interest (Block 50):
    - User claims accrued interests.
    - `markets[vToken].rewardIndex` updated via `accrueInterest`.
    - Over 15 blocks, 120 USDT income with score change (sum of scores becomes 60).
    - `markets[vToken].rewardIndex` increases by +2.
6. Interest Calculation for the User:
    - User's interest calculated using `_interestAccrued`.
    - The difference between `markets[vToken].rewardIndex` (2.7) and `interests[market][account].rewardIndex` (0.7) is 2 USDT per score point.
    - Multiplied by user's score (10) equals 20 USDT accrued interest.
7. Updating User's Reward Index:
    - `interests[vToken][user].rewardIndex` set to the new `markets[vToken].rewardIndex`.

### Prime

![https://res.cloudinary.com/duqxx6dlj/image/upload/v1696427287/VenusPrime/Untitled_2_czwzkp.png](https://res.cloudinary.com/duqxx6dlj/image/upload/v1696427287/VenusPrime/Untitled_2_czwzkp.png)

- `Prime.sol` is a critical component of Venus Protocol, offering users the opportunity to earn rewards generated from specific markets within the protocol. Here's how it works:
    - To be eligible for a Prime token, regular users must stake a minimum of 1,000 XVS for a continuous period of 90 days.
    - Once this condition is met, users can claim their Prime token, signaling the beginning of rewards accrual.
    - Prime token holders have the flexibility to claim their accrued rewards at their convenience.

**Key Features and Functions in `Prime.sol`**:

- Utilizes the Cobb-Douglas function to calculate user rewards.
- Offers two types of Prime Tokens:
    - Revocable: Can be minted after staking a minimum of 1,000 XVS for 90 days but can be burnt if the stake falls below 1,000 XVS.
    - Irrevocable (Phase 2) - Specifics for this token type will be detailed in Phase 2 of the Venus Prime program.

### **PrimeLiquidityProvider**

![https://res.cloudinary.com/duqxx6dlj/image/upload/v1696427286/VenusPrime/Untitled_3_f6zbiq.png](https://res.cloudinary.com/duqxx6dlj/image/upload/v1696427286/VenusPrime/Untitled_3_f6zbiq.png)

- `PrimeLiquidityProvider.sol` serves as the second source of tokens for the Prime program, complementing the tokens generated by Venus markets. Key details include:
    - This contract, `PrimeLiquidityProvider`, allocates a predetermined quantity of tokens to Prime holders in a uniform manner over a defined period.

**Key Features and Functions in `PrimeLiquidityProvider.sol`**:

- Ability to add new tokens for rewards accrual.
- Configurable token distribution speed.
- Safeguard mechanisms for accidentally sent tokens.
- Ability to pause and resume token transfers, ensuring control and stability within the program.

## Architecture Recommendations

- **Staking and Earning Venus Prime Tokens**: These are essential parts of the Venus Prime Yield Protocol. Let's break them down and talk about ways to make them even better:
- **Staking as it is now**:
    - Right now, if you want to earn a Prime Token, you have to lock up at least 1,000 XVS for 90 days. This Prime Token represents the XVS you've staked and comes with some perks. But if you take out some XVS and drop below 1,000, you'll lose your Prime Token.
- **Ways to Make Staking Better**:
    - **Different Staking Levels**: Instead of one fixed requirement, we could have different levels for staking. This way, even if you have fewer tokens, you can join in. Different levels might offer different bonuses to encourage more staking.
    - **Flexible Staking Time**: We could let you choose how long you want to stake your XVS, depending on what suits you. Staking for longer could mean more rewards, which could motivate people to keep their XVS locked up longer.
    - **Smart Staking Rules**: The rules for how much you need to stake could change depending on how the market is doing and how much XVS is worth. This could help keep things steady and reliable.
- **What You Can Do with Venus Prime Tokens**:
    - **More Ways to Use Them**: We could give you more things to do with your Prime Tokens, so they're even more useful in the Venus ecosystem. That could encourage more people to join in.
    - These tokens are like keys to the Venus ecosystem. You can use them to vote on changes, earn rewards, and access different parts of the system.
- These changes could make staking easier and more rewarding, and make Venus Prime Tokens more valuable. It's all about making the system better for everyone.

## **Centralization Risks**

- **Smart Contract Centralization**: The protocol is based on smart contracts, which are autonomous and decentralized by nature. However, the developers of the protocol have the ability to update or modify these contracts, introducing a potential point of centralization. If these powers are abused or misused, it could lead to centralization risks
- **Governance Centralization**: The governance of Venus Prime is controlled by XVS token holders. If a small group of individuals or entities comes to own a large portion of XVS tokens, they could potentially control the governance of the protocol, leading to centralization. This can include making decisions that favor them at the expense of other users

## Systemic Risks

1. **Governance Risk**: The protocol faced a hacking attempt where the attacker tried to gain control of the protocol by bribery (VIP42). Although the attempt was thwarted, it highlights the risk of governance attacks
2. **Risk Fund Adequacy**: Venus has a risk fund established to address potential shortfalls in the protocol, particularly in situations of ineffective or delayed liquidations. However, if the fund is not adequate to cover a major event, it could potentially result in a systemic risk
3. **Price Oracle Resilience**: Venus V4 introduces resilient price feeds. If these feeds fail or provide inaccurate data, it could potentially destabilize the system. The protocol's resilience to such an event is yet to be tested
4. **Dependence on XVS Staking**: Venus Prime requires users to stake at least 1,000 XVS for 90 days to mint their Prime Token. If a user decides to withdraw XVS and their balance falls below 1,000, their Prime Token will be automatically revoked. This introduces a risk if there's a significant drop in the value of XVS or if a large number of users decide to unstake and sell their XVS simultaneously

## Key Takeaways

1. **Dual Token System:** The protocol introduces a bifurcated token system, comprising revocable and irrevocable Prime tokens. Each token variant carries its unique rules and benefits, offering users a versatile array of choices.
2. **Sustainable Rewards:** Diverging from conventional incentive models dependent on external sources, Venus Prime generates its rewards intrinsically within the protocol. This inherent mechanism not only fosters sustainability but also augments the potential for long-term growth.
3. **Long-Term Commitment:** Users are incentivized to uphold a commitment to the protocol by staking XVS tokens for a minimum duration of 90 days. This prolonged engagement serves the dual purpose of fostering dedication and dissuading premature withdrawals.
4. **Complex Reward Calculation:** Venus Prime employs a sophisticated reward calculation formula known as the Cobb-Douglas function to ascertain user rewards. This intricacy, while daunting on the surface, is intricately designed to uphold principles of equity and precision in reward distribution.

### Time spent:
45 hours