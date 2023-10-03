# Analysis - Venus Prime 
# Summary

| List |Head |Details|
|:--|:----------------|:------|
|1 | Overview of Venus Prime   | overview of the key components and features of Venus Prime    |
|2 |Audit approach | Process and steps I followed  |
|3 |Learnings | Learnings from this protocol|
|4 |Possible Systemic Risks | The possible systemic risks based on my analysis |
|5 |Code Commentary | Suggestions for existing code base |
|6 | Centralization risks | Concerns associated with centralized systems |
|7 |Risks as per Analysis | Possible risks |
|8 |Non-functional aspects | General suggestions |
|9 |Time spent on analysis  | The Over all time spend for this reports |

## Overview

``Venus Prime`` is a revolutionary incentive program aimed to bolster user engagement and growth within the 
protocol. Venus Prime aims to enhance yields and promote $XVS staking, focusing on markets including USDT, 
BNB, BTC and ETH.

### ``Venus Prime essentials``
Venus Prime's uniqueness lies in its self-sustaining rewards system, instead of external sources, rewards are derived from the protocol's revenue, fostering a sustainable and ever-growing program.

Eligible $XVS holders will receive a unique, non-transferable Soulbound Token, which boosts rewards across selected markets.

#### ``Prime tokens``
Venus Prime encourages user commitment through two unique Prime Tokens:

1. Revocable Prime Token:

Users need to stake at least 1,000 XVS for 90 days in a row.
After these 90 days, users can mint their Prime Token.
If a user decides to withdraw XVS and their balance falls below 1,000 XVS, their Prime Token will be automatically revoked.

2. Irrevocable "OG" Prime Token (Phase 2):

It is yet to be defined in the future.

``Venus Prime`` aims to incentivize larger stake sizes and diverse user participation. This is expected to significantly increase the staking of XVS, the Total Value Locked (TVL), and market growth.

``Venus Prime`` intends to promote user loyalty and the overall growth of the protocol. By endorsing long-term staking, discouraging premature withdrawals, and incentivizing larger stakes, Venus Prime sets a new course in user engagement and liquidity, contributing to Venus Protocol's success.

User needs to Stake their $XVS tokens to be eligible for Venus Prime.

### ``Rewards Mechanism``
``Venus Prime`` is using ``Cobb-Douglas`` function to calculate scores and rewards for users,the Cobb–Douglas production function is a particular functional form of the production function, widely used to represent the technological relationship between the amounts of two or more inputs (particularly physical capital and labor) and the amount of output that can be produced by those inputs.

Reward Formula: Cobb-Douglas function

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

**Qualifiable XVS Staked:**

$$
\tau_i =
\begin{cases}
\min(100000, \tau_i) & \text{if } \tau_i \geq 1000 \\
0 & \text{otherwise}
\end{cases}
$$

**Qualifiable supply and borrow:**

$$
\begin{align*}
\sigma_{i,m} &= \min(\tau_i \times borrowMultiplier_m, borrowedAmount_{i,m}) \\
&+ \min(\tau_i \times supplyMultiplier_m, suppliedAmount_{i,m})
\end{align*}
$$

#### Significance of α
A higher value of α increases the weight on stake contributions in the determination of rewards and decreases the weight on supply/borrow contributions. The value of α is between 0-1

A default weight of 0.5 weight has been evaluated as a good ratio and is not likely to be changed. A higher value will only be needed to attract more XVS stake from the Prime token holders at the expense of supply/ borrow rewards.
    
#### Income collection and distribution
The `Prime` contract takes the total released + unreleased funds and distributes to Prime token holders each block. The distribution is proportional to the score of the Prime token holders.

When a user claims their rewards and if the contract doesn’t have enough funds then Prime triggers the release of funds from PSR to `Prime` contract in the same transaction i.e., in the `claim` function.

 
## Audit approach

I followed below steps while analyzing and auditing the code base.

1. Read the contest Readme.md and took the required notes.

  - Venus Prime
    - Composition
    - Test Coverage - 96%
    - Venus Prime is a new token claimable by users who satisfy some constraints in terms of XVS staked (we'll upgrade the XVSVault implementation) and interactions with the markets (we'll upgrade the Comptroller implementation of the Venus Core pool)
    - Multi-Chain
    - Uses oracle under the hood
    - smart contracts will be deployed on  BNB Chain, Ethereum mainnet, Arbitrum, Polygon zkEVM, opBNB
      
2. Analyzed the over all codebase one iterations very fast

3. Study of documentation to understand each contract purposes, its functionality, how it is connected with other contracts, etc.

4. Then i read old audits and already known findings. Then went through the bot races findings 

5. Then I started to go through my second iteration and this time function by function and line by line understanding the whole protocol deeply and took the necessary notes to ask some questions to sponsors.


## Stages of audit

- ``The first stage of the audit``

During the initial stage of the audit for Venus Prime I focused on understanding the code and getting familiar with the code and finding any low hanging issues in the codebase of Venus Prime, also this stage of my audit let me focus on finding QA's or GAS optimisation issues in the codebase of Venus prime.

- ``The second stage of the audit``

In the second stage of the audit for Venus Prime my main focus shifted from finding the low hanging bugs to finding more protocol related bugs or logic related bugs and also understanding the code of Venus Prime in depth, This involved identifying and analyzing the importand functions in the Prime.sol contract and PrimeLiquidityProvider.sol contract. By examining these two contracts the audit aimed to gain a comprehensive understanding of the protocol's functionality and potential risks.


- ``The third stage of the audit``

During the third stage of the audit for Venus Prime, my main focus was thoroughly examining and marking any doubtful or vulnerable areas within the protocol. This stage involved conducting comprehensive vulnerability assessment and identifying potential weaknesses in the system. I found many ``vulnerable`` code parts and marked them with ``@audit tags``.


- ``The fourth stage of the audit``

During the fourth stage of the audit for Venus Prime, a comprehensive analysis and testing of the previously identified doubtful and vulnerable areas were conducted. This stage involved diving deeper into these areas, performing in-depth examinations, and subjecting them to rigorous testing, including fuzzing with various inputs. Finally concluded findings after all research's and tests. Then i reported C4 with proper formats 




## Centralization risks

A single point of failure is not acceptable for this project Centrality risk is high in the project, the role of ``onlyOwner``detailed below has very critical and important powers

Wrong Prime token may be set by a malicious or stolen private key ``onlyOwner`` ``msg.sender``

```solidity

File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

```function setPrimeToken(address prime_) external onlyOwner {
        _ensureZeroAddress(prime_);

        emit PrimeTokenUpdated(prime, prime_);
        prime = prime_;
    }
```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118-L126

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

   function initializeTokens(address[] calldata tokens_) external onlyOwner {
        for (uint256 i; i < tokens_.length; ) {
            _initializeToken(tokens_[i]);

            unchecked {
                ++i;
            }
        }
    }


```
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L225

```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
        uint256 balance = token_.balanceOf(address(this));
        if (amount_ > balance) {
            revert InsufficientBalance(amount_, balance);
        }

        emit SweepToken(address(token_), to_, amount_);

        token_.safeTransfer(to_, amount_);
    }
```

## Code Commentary

### General Code suggestions for all contracts

1. Use more recent version of solidity
2. Some checks can use modifiers 
3. Hard coded values should be avoided
4. Add more detailed comments to explain complex logic, calculations, and the purpose of functions.


## Non-functional aspects

- Aim for high test coverage to validate the contract's behavior and catch potential bugs or vulnerabilities
- The protocol execution flow is not explained in efficient way. 
- Its best to explain over all protocol with architecture is easy to understandings 

## Time spent on analysis 
``15 Hours``

### Time spent:
15 hours

### Time spent:
15 hours