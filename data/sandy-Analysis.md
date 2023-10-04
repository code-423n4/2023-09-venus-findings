# Venus Protocol - Analysis 

|Head |Details|
|:----------------|:------|
|Description| A brief introduction to the protocol
| Approach taken in evaluating the codebase | What is unique? How are the existing patterns used? |
|Codebase quality analysis| Its structure, readability, maintainability, and adherence to best practices|
|Architecture recommendations| The design of the protocol|
|Centralization risks| power, control, or decision-making authority is concentrated in a single entity|
|Other recommendations| Recommendations for improving the quality of your codebase|
|Conclusion| Takeaway from this protocol, recommendations and final review|
|Time spent| Total time spent during auditing and reviewing the codebase |

## Description
Venus Prime, a revolutionary incentive program aimed to bolster user engagement and growth within the protocol. An integral part of [Venus Tokenomics v3.1](https://docs-v4.venus.io/governance/tokenomics), Venus Prime aims to enhance rewards and promote $XVS staking, focusing on markets including USDT, USDC, BTC and ETH. The launch is targeted for early Q4 2023.

The key contracts of the protocol for this Audit are:
1. Prime.sol: Soulbound token that will allow holders to accrue rewards, generated with part of the income of some markets in the Venus Protocol. Regular users have to stake 1,000 XVS at least during 90 days to be eligible for a Prime token, that users will be able to claim as soon as they satisfy the constraint. After claiming their Prime token, the rewards start to be accrued and Prime holders will be able to claim them when they want.

2. PrimeLiquidityProvider: The second source of tokens for the Prime program (the first one are the Venus markets) will be this contract: PrimeLiquidityProvider. It will allow to define a fixed amount of tokens to be distributed uniformly to the Prime holders for a period of time.

3. Libs Scores, FixedMath and FixedMath0x: Used in the calculations needed to accrue rewards for Prime holders.



## Approach taken in evaluating the codebase

Steps:

- ``Using a static code analysis tool``: Static code analysis tools can scan the code for potential bugs and vulnerabilities. These tools can be used to identify a wide range of issues, including:

    - Insecure coding practices
    - Common vulnerabilities
    - Code that is not compliant with security standards

- ``Reading the documentation``: The [documentation](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/README.md) was perfectly crafted by the Venus Protocol team explaining all the necessary components and mechanisms of the code base with examples. This [documentation](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/README.md) helped to provide a detailed overview of the protocol and its code base. It also helped to to understand the purpose of the code and to identify potential areas of concern.

- ``Scoping the analysis``: Once you have a basic understanding of the protocol and its codebase, you can start to scope the analysis. This involves identifying the specific areas of code that you want to focus on. For example, you may want to focus on the code that handles user input, the code that interacts with external calls, or the code that changes sensitive state.

- ``Manually reviewing the code``: Once you have scoped the analysis, you can start to manually review the code. This involves reading the code line-by-line and looking for potential problems. Some of the things you should look for include:

   - Unvalidated user input
   - Unsafe external calls
   - Unexpected state changes
   - Lack of Access Control

- ``Marking vulnerable code parts with @audit tags``: Once you have identified any potential vulnerabilities, you should mark them with @audit tags. This will help you to identify the vulnerable code parts later on. 

- ``Digging deep into vulnerable code parts and compare with documentations``:  For each vulnerable code part, you should dig deep to understand how it works and why it is vulnerable. You should also compare the code with the documentation to see if there are any discrepancies.

- ``Performing a series of tests``: Once you have finished reviewing the code, you should perform a series of tests to ensure that it works as intended. These tests should cover a wide range of scenarios, including:

  - Valid and invalid user input
  - Different types of attack vectors
  - Complex state changes

- ``Reporting any problems found``:  If you find any problems with the code, you should report them to the developers of Venus protocol. The developers will then be able to fix the problems and release a new version of the protocol.

## Codebase quality analysis
There were 7 contracts in scope for this Audit with total nSloc of 1039 and 7 external imports. Hence, the code base was pretty small. But this code base packed a lot of advanced functionalities. It was nice to see 96% test coverage for the code as this is a very complex protocol. All the complex functionalities were explained with NatSpec comments and thus, the quality of the code base is pretty good. There is a need to understand a separate part of the codebase / get context in order to audit this  the protocol but it was also well documented and easy to comprehend. This is the 5th audit for this protocol and thus the code base is pretty secure. This protocol also uses oracle but more securely. Under the hood this is an extra layer on top of Chainlink, Binance oracle, Pyth network and TWAP oracle, allowing the comparison of values returned to decide if they are valid or not. This code base also adheres to all the best practices like using name imports, custom errors, indexed event parameters, view and pure functions, etc. Thus, this is a very high quality code base.

## Architecture recommendations
This code base is pretty simple architecture wise as there are not much components to interact for the end user. But, this is not a standalone code base and various other parts of the Venus Protocol interact with this protocol. 

The User only interacts with ``claim()`` function to claim the prime token after 90 days staking of 1000 XVS is completed. And after claiming the prime token, the user only interacts with ``claimInterest()`` function to claim the rewards for the obtained prime token. 

But, the architecture is not as simple as this. Other contracts like ProtocolShareReserve.sol and PrimeLiquidityProvider.sol are the contracts that will provide funds to the Prime.sol contract for distribution as per the demand. The XVSVault.sol is one of the most important contract that calls XVSUpdated on Prime.sol contract which helps to review and fulfill the necessary requirement for holding prime token. Also, the PolicyFacet.sol needs to execute ``accrueInterestAndUpdateScore()`` on Prime.sol contract after executing any operation that could impact the Prime score or interest. Thus, there are lot of components interacting with each other which makes the protocol more complex for developers but the user experience with interaction with the protocol is pretty smooth and easy. 


## Centralization risks
Owners and admins are the Normal timelock contract, that is part of the Governance protocol.

Regarding authorization the `AccessControlManager` (ACM) deployed at https://bscscan.com/address/0x4788629abc6cfca10f9f969efdeaa1cf70c23555 is used.

In this ACM, only [0x939bd8d64c0a9583a7dcea9933f7b21697ab6396](https://bscscan.com/address/0x939bd8d64c0a9583a7dcea9933f7b21697ab6396) (Normal timelock) has the DEFAULT_ADMIN_ROLE. And this contract is a Timelock contract used with the Venus Improvement Proposals.

There are two other Timelock contracts to execute VIP's with a shorter delay:

* Normal: 24 hours voting + 48 hours delay
* Fast-track: 24 hours voting + 6 hours delay
* Critical: 6 hours voting + 1 hour delay

Thus, the proper implementation of time lock for owners and admins really help to minimize centralization risk for this protocol.

## Other recommendations

- Regular code reviews and adherence to best practices
- Conduct external audits by security experts
- Consider open sourcing the contract for community review
- Maintain comprehensive security documentation
- Establish a responsible disclosure policy for vulnerabilities
- Implement continuous monitoring for unusual activity
- Educate users about risks and best practices

## Conclusion
Venus protocol introduces Venus Prime which is a new token claimable by users who satisfy some constraints in terms of XVS staked and interactions with the markets).The code base is very secure, well-written and test coverage is also very good. It is the first protocol that I audited which boosted yield with the help of Cobb-Douglas function, inspired by the Goldfinch rewards mechanism.

## Time-spent 

40 Hours 

### Time spent:
40 hours