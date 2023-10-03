
# Description

The Venus Prime program is designed to incentivize liquidity providers and borrowers to stake XVS. This is achieved through the use of the Prime token. To participate in the Prime program, you need to:
1. **Have some amount staked or borrowed with Venus.**
2. **Stake at least 1000 XVS for at least 90 days.**

Once these conditions are met, Venus will incentivize you with consistent rewards in the markets you participate in (borrowing/supplying).

The program primarily functions with two methods:
1. **Score:** This represents each user's individual supply and balance.
2. **Interest:** This denotes the rewards generated for every 1 point of score.

Through these two methods, the Prime program encourages liquidity providers and borrowers to purchase and stake XVS, thereby increasing its price and overall value in the meantime.

| Severity | Occurrences |
|----------|-------------|
| High     | 3           |
| Medium   | 3           |
| Low      | 6           |
| Gas      | 3           |
| Analisys | 6 hours     |


# Time allocations

|            |            |
|------------|------------|
| Start date | 28.09.2023 |
| End date   | 03.10.2023 |
| **Total**  | **5 days** |

| Members       | Positions            | Time spent |
|---------------|----------------------|------------|
| **0x3b**      | full time researcher | ~30H       |


# Findings
The codebase was "a blast," and the time spent was just perfect. I had enough time to delve into every aspect of the scope, even exploring some out-of-scope codebases to gather reference points. The economics were truly fascinating, and I examined them from every perspective. In fact, approximately 80% of my time was dedicated to attempting to manipulate the rewards, user's score, and interest. I encountered several scenarios where any user could potentially cause damage and extract value. Although most of the bugs that I found were in the Yearly APR calculations, which are view functions used only for the front end. Even though no funds were at risk, these functions were quite buggy and could cause some issues with the calculations and the overall system.

# Codebase quality
The code quality was excellent, having undergone four previous audits, one of which was conducted by OZ. However, it contained some unnecessary functions that only contributed to increased gas consumption during deployments. The scoring and interest components were highly sophisticated, but due to numerous missing pieces of information related to how vTokens work, I found myself investing more time than intended in out-of-scope code exploration and extensive documentation reading.

1. **Code Comments**
Most comments were good, but there's always room for improvement. In some parts, especially in the Interface contracts, making comments and explanations clearer would help developers work with the code more easily. Adding more detailed comments and documentation for complex or important parts of the code can make the whole codebase easier to understand and maintain. This would benefit not only the current development team but also make it simpler for future contributors to work with the existing code.

2. **Documentation**
The documentation was well-crafted, providing a comprehensive explanation of the system. The mathematical equations were the exceptional feature, as they provided insight into how the system operates beneath the surface.

3. **Organization**
Codebase is very mature and well organized with clear distinctions between the 6 contracts, and their respective interfaces.


4. **Testing**
- **Unit Tests:** The unit tests were sophisticated enough to uncover basic bugs. However, I have noticed a few issues with them:

    1. There were not enough unit tests, especially testing the functions from only one or two different angles.
    2. Most of them were full end-to-end function tests and tests for user-expected paths, lacking code-breaking tests. 
    3. The use of mocks was excessive. They were used too frequently, which is not considered best practice. Mocks are often simpler and sometimes inaccurate code representations. I would recommend using mocks **only** for things that cannot be represented by your contracts, such as oracles (e.g., Chainlink or Balancer) and other entities (e.g., CRV or UNI).
    4. The main issue was that deployed vTokens have 8 decimals, whereas the mocks for the vTokens used in the tests have 18. This led to several issues, including one or two that I have categorized as medium and high severity.
    5. I was not able to see any fuzz or invariant test. However my knowage with hardhat is "not amazing" and I might have just missed them.



###### Prime.sol
The Prime contract is the main contract for the update, handling all updates related to scores and interest in the Prime program. It is accompanied by Protocol Shared Reserve (PSR) and Prime Liquidity Provider (PLP), which are two contracts that manage external income. Under Prime, there are several implemented features:

1. Prime token - Users must stake a minimum of 1000 XVS for at least 90 days to participate.
2. Score accounting - This system manages each individual's contributions to the system, including their supply and borrow amounts.
3. Interest accounting - This rewards users of the Prime program based on the total income generated.
4. Markets - The DAO can add multiple markets representing different tokens, such as vUSDC, vUSDT, vETH, vBTC, and more.
5. Interest claims - Users can use a few functions to claim their generated interest, calculated based on "interest * score". For example, if 1 USDT is generated per interest, and you have a score of 10, you can claim 10 USDT.
6. APR math - This involves mathematical calculations for the yearly APR, used for front-ends to display the yearly APR and help users predict their income.
7. Prime Irrevocable token - This is similar to a Prime token, but it cannot be removed and remains with you indefinitely. It's not fully implemented yet, as there are no criteria for how to upgrade to it.

With the mechanisms mentioned above, "Prime.sol" controls the Prime incentives and is used for the main user flow.

To participate, you will need to:

1. Supply or borrow some tokens with Venus.
2. Stake at least 1000 XVS for 90 days.

###### PrimeStorage.sol
Prime Storage is the storage contract used by [Prime](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol). There are no functions inside it; it contains only regular storage. It includes the storage of some global variables, mappings for scores and interest, as well as structs to manage users and markets.

###### PrimeLiquidityProvider.sol
Prime Liquidity Provider is the contract helper of [Prime](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol), as it's one of the two contracts that can supply Prime with tokens (the other one being PSR, although it is out of scope). With this contract:

1. The DAO can initialize prime tokens.
2. Accrue funds - Calculate the internal balances and assign a certain value to be sent later to [Prime](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol) when [releaseFunds](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L192-L205) is called.
3. Release funds - When [Prime](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol) is in need of tokens, this function is called. It sends the stored tokens to [Prime](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol).

###### Scores.sol
Scores is the math contract used to calculated a given user's score. It uses a special math function called - **Cobb-Douglas** - which is described in the docs as:

$$
Rewards_{i,m} = \Gamma_m \times \mu \times \frac{\tau_{i}^\alpha \times \sigma_{i,m}^{1-\alpha}}{\sum_{j,m} \tau_{j}^\alpha \times \sigma_{j,m}^{1-\alpha}}
$$

Where:

- $`Rewards_{i,m}`$ = Rewards for user $`i`$ in market $`m`$
- $`\Gamma_m`$ = Protocol Reserve Revenue for market $`m`$
- $`μ`$ = Proportion to be distributed as rewards
- $`α`$ = Protocol stake and supply & borrow amplification weight
- $`τ_{i}`$ = XVS staked amount for user $`i`$
- $`\sigma_i`$ = Sum of **qualified** supply and borrow balance for user $`i`$
- $`∑_{j,m}`$ = Sum for all users $`j`$ in markets $`m`$


###### FixedMath.sol
FixedMath involves some simplified yet accurate mathematical calculations. It is used to multiply and divide values while ensuring that the inputs are correct according to the formula being used.

###### FixedMath0x.sol
FixedMath0x is a more complex version of FixedMath. It is used to calculate intricate mathematical problems, such as the "natural logarithm of a fixed-point number" and the "natural exponent for a fixed-point number," which are challenging to compute using regular code. That's why [FixedMath0x](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol) primarily employs assembly bit representation of numbers.


# Architecture recommendations


The contracts provided to us were sufficient to achieve the system's intended goals. They included interest acrural, user scores and (mostly) math calculators. They were relatively simple, and the code was straightforward, making the audit a pleasant experience. 

**Positive Suggestions:**
1. One potential enhancement I recommend is implementing a flywheel mechanism to further incentivize and encourage users to stake on Venus.
2. Another alternative worth considering is expanding on the NFT concept. Users who have achieved Prime status could claim it as an NFT and sell it on an open market like OpenSea, along with their XVS balance inside. This approach could introduce an intriguing dimension to the system and offer users additional benefits for their participation in Venus. This way, users who don't have the patience to wait 90 days but have the extra money to spend can buy an NFT holding 1000 XVS, for a premium, of course, since they didn't have to wait 90 days.


**Some things that may warrant a second look from the devs:**
1. **Reduce the use of mocks** - I would recommend minimizing the use of mocks to only what is necessary. Sometimes, mocks oversimplify the code, while at other times (e.g., tests for Prime with vTokens having 18 decimals), they are incorrect and can lead to the emergence of bugs in the code.
2. **Include inv and fuzz tests** - These are essential for a comprehensive system.
3. **Market removal** - I suggest adding a function to remove unnecessary markets. This is because most accrual functions rely on a for loop of all markets to accrue rewards, and having more markets results in higher gas costs.

# Centralization risk

Venus demonstrates an exceptionally high level of decentralization. This decentralization paves the way for a brighter future where all users are empowered to shape the ecosystem as they see fit. 

However, on the other note, centralization may not be a major concern, provided that these admins are genuinely invested in the project's overall growth. When harnessed effectively, centralization can provide the system with a competitive edge over its competitors. It facilitates rapid decision-making, enabling Venus to adapt swiftly and stay ahead of industry trends. Making it less of a corporate gian and ore of a 3 man start-up. In contrast, decentralized DAO systems often require proposals, votes, and lengthy approval and implementation processes.

Nevertheless, it's crucial to acknowledge that centralization also carries potential downsides, such as an increased vulnerability to single points of failure. Striking a balance between centralization and decentralization is important to ensure the system's robustness and sustainability.

# Systemic risks
The current state of this protocol leaves it susceptible to various types of economic attacks. Certain functions within the code can be manipulated by users to gain advantages over others or even illicitly acquire tokens from the protocol. However, after a comprehensive refinement following this audit, the risks associated with these vulnerabilities can be significantly reduced. By addressing these weaknesses and implementing robust security measures, the protocol can become more resilient against potential economic attacks.

Nonetheless, it is crucial to exercise caution when adding new features to the code, especially those involving economic incentives. While they can enhance the protocol's performance and attract users, they also tend to be the most vulnerable points, susceptible to exploitation if not designed and implemented with the utmost care.


**Some potential risks may include:**
1. **Weird ERC20 tokens and their associated markets:** With the addition of markets, the DAO needs to exercise caution when deciding what to include and what to exclude.
2. **Price volatility:** When calculating scores, the price of a given vToken is also taken into account, meaning that rapid market changes can lead to significant disruptions in the ecosystem.
3. **The system's requirement to accrue users:** This may be unnecessary, as it results in the wastage of capital on gas. This capital could otherwise be used by Venus to enhance rewards, the market itself, or invest in further project development.


# Conclusion
In general, the Venus project exhibits an interesting and well-developed architecture I believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

### Time spent:
30 hours