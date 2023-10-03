# Analysis

## Summary

| Count | Topic | 
|:--:|:-------|
| 1 | Introduction |  
| 2 | High Level Contract Flow |  
| 3 | Audit Approach |  
| 4 | Codebase Quality |   
| 5 | Architecture Recommendation |  
| 6 | Centralization Risk |
| 7 | Systemic Risks |
| 8 | Code Review Insights |
| 9 | Time Spent |

## Introduction

This technical analysis report examines the security aspects of Venus Prime, a vital component of the Venus Protocol's incentive program. The analysis explores the underlying smart contracts and different components, including reward distribution calculations, claiming mechanism, and unique Soulbound Tokens, while identifying security insights and potential vulnerabilities within the Venus Prime contracts.

## High Level Contract Flow

![High Level Contract Flow](https://res.cloudinary.com/davyfibzy/image/upload/v1696261747/nzz6hhgqd3edr6l3fslz.png)

## Audit Approach

1. **Initial Scope and Documentation Review**: Thoroughly examined the Contest Readme File and Venus Prime Readme file to understand the contract's objectives and functionalities.
2. **High-level Architecture Understanding**: Performed an initial architecture review of the codebase by skimming through all files without exploring function details.
3. **Test Environment Setup and Analysis**: Set up a test environment and executed all tests.
4. **Comprehensive Code Review**: Conducted a line-by-line code review focusing on understanding code functionalities. 
    * **Understand Codebase Functionalities**: Began by comprehending the functionalities of the codebase to gain a clear understanding of its operations.
    * **Analyze Value Transfer Functions**: Adopted an attacker's mindset while inspecting value transfer functions, aiming to identify potential vulnerabilities that could be exploited.
    * **Identify Access Control Issues**: Thoroughly examined the codebase for any access control problems that might allow unauthorized users to execute critical functions.
    * **Detect Reentrancy Vulnerabilities**: Looked for potential reentrancy attacks where malicious contracts could exploit reentrant calls to manipulate the contract's state and behavior.
    * **Evaluate Function Execution Order**: Examined the sequence of function executions to ensure the protocol's logic cannot be disrupted by changing the order of calls.
    * **Assess State Variable Handling**: Identified state variables and assessed the possibility of breaking assumptions to manipulate them into exploitable states, leading to unintended exploitation of the protocol's functionality.

5. **Report Writing**: Write Report by compiling all the insights I gained throughout the line by line code review.


## Codebase Quality

| Codebase Quality Categories  | Comments |
| --- | --- |
| *Unit Testing*  | The codebase has undergone extensive testing. |
| *Code Comments*  | In general, the code had comments for what each function does, which is great. However, it could be better if we also added comments for the custom data types (struct) used in the code. This would help make it clearer how each data type is used in the contract. |
| *Documentation* | Prime Documentation was to the point, containing all the major details about how the contract is expected to perform. |
| *Organization* | Code orgranization was great for sure. The flow from one contract was perfectly smooth. |

## Architecture Recommendation

* `Block.number Vs Block.timestamp`: Since the contracts are planned for deployment on multiple chains, including those like Optimism where `block.number` may not accurately represent time, it is advisable to use `block.timestamp` consistently. In the `PrimeLiquidityProvider` (PLP) contract, variations in average block time can lead to fluctuations in the rate at which tokens accrue.

* `Solidity Version Used`: Consider upgrading the Solidity version from 0.8.13 to 0.8.15 or a higher version. Although the current version does not impact any contracts within scope, it's a important to avoid any potential issues which 0.8.13 version brings related to ABI-encoding and optimizer bugs regarding memory side effects of inline assembly.

* `Handling Multiple Markets with Same Underlying Token`: Address the limitation in the Prime contract, which currently cannot handle multiple markets with the same underlying token. This issue could have long-term implications, especially if multiple markets with identical underlying tokens need to be added to the `Prime` program simultaneously. It's recommended to enhance the contract's ability to handle such scenarios for improved flexibility and scalability.

## Centralization Risk

* `Token Sweep Power`: The owner has the authority to sweep all tokens held in the PrimeLiquidityProvider contract. This centralized control could potentially pose serious risk.

* `Claim Functionality Pause`: The owner retains the capability to temporarily "pause" the claim functionality whenever necessary. This centralized ability to halt operations could affect user access and functionality.

* `Privileged Role Impact`: Privileged roles have the ability to modify the values of `alpha` and `multiplier`, which can have a substantial impact on the interest earned by users.

## Systemic risks

* `Sybil Risk`: The contract employs a cap named `MAXIMUM_XVS_CAP` to prevent users from earning interest on amounts exceeding this limit. However, in practice, a user can potentially exploit this by using multiple accounts to accumulate more interest. As a result, the existing protection measures in place are not entirely immune to Sybil attacks.

* `DoS in Key Functionities`: `sweepToken` function is intended to recover accidentally transferred ERC-20 tokens to the contract. While this function is restricted to the contract owner (`onlyOwner` modifier), it does not account for previously accrued tokens, which poses a significant risk to the protocol's stability. In case it sweeps the token which are previously added to total accrued tokens but not released to Prime yet, then it will lead to DoS across both this contracts.

* `Gas Limit Issues`: There is no way to update Max Loops value in Future. While `addMarket` controls the number of markets that can be add so it not that big a risk. But still as there is no way to even remove the market, it is a possible risk if block limit is reached.

## Code Review Insights

**Key Contracts:**

1. `Prime.sol`: Soulbound token, the central contract of this audit, allows Prime holders to accumulate rewards, which they can later claim at their convenience.

2. `PrimeLiquidityProvider.sol`: This contract offers liquidity to the Prime token consistently over a specific duration. This liquidity serves as one of the two fund sources supported by Prime tokens.

### Insights

* The rate at which tokens accumulate in `PrimeLiquidityProvider` may not be consistent across different blockchain networks.

* The `sweepToken` function has the potential to sweep already accrued rewards, which could result in Denial-of-Service (DoS) vulnerabilities and financial losses for users.

* There is a risk of encountering an infinite loop within the `updateScores` function.

* There is a decimal precision error which is leading to incorrect calculation of Score.

* The value of `BLOCKS_PER_YEAR` may be unpredictable during deployment, particularly on blockchain networks with inconsistent block times.

* Tokens that have fees associated with them may lead to exaggerated interest claims for some users, potentially causing issues for the remaining users.

* Irrevocable tokens can be burned.

* There is currently no mechanism in place to update the maximum loop count in the future.

* The `issue` function allows the issuance of Prime Tokens to users without enforcing the 90-day staking condition.

* The `Prime` contract does not support the use of multiple markets with the same underlying tokens.

## Time Spent

| Total Number of Hours | 16 |
|:--:|:--:|




### Time spent:
16 hours