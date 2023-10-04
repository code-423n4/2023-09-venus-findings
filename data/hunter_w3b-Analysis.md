![Good Entry](https://www.gitbook.com/cdn-cgi/image/width=256,dpr=2,height=40,fit=contain,format=auto/https%3A%2F%2F1450767434-files.gitbook.io%2F~%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSfxZmzf9N5T5Npv859XY%252Flogo%252FfhLLbUOJcE1YdxFsX1pJ%252FVenus%2520design.png%3Falt%3Dmedia%26token%3Da75c0f1b-3c34-44ff-82a5-e10b8ab795ec)

# Analysis Venus Prime Contest

## Description

Venus Protocol introduces Venus Prime, an innovative incentive program within Venus Tokenomics v3.1. Venus Prime enhances rewards and encourages $XVS staking in markets like USDT, USDC, BTC, and ETH. What makes it unique is its self-sustaining rewards system, driven by protocol revenue. Eligible $XVS holders receive a non-transferable Soulbound Token to boost rewards in selected markets.

**The key contracts of the protocol for this Audit are**:

- **Prime.sol**:
  This is the main contract for the Prime token. It's responsible for handling rewards distribution to Prime holders and allowing them to claim their rewards at their convenience.

  - **Libraries Used**:
    SafeERC20Upgradeable.sol from OpenZeppelin: For safe handling of ERC20 tokens.
    PausableUpgradeable.sol from OpenZeppelin: For pausing contract operations when necessary.
    MaxLoopsLimitHelper.sol (from an external repository): Potentially used for limiting loops to prevent excessive gas usage.
    AccessControlledV8.sol from Venus Protocol: Used for access control management.
    OracleInterface.sol from Venus Protocol: Used for interacting with oracles.

- **IPrime.sol**:
  This is an interface for the Prime contract, defining the contract's functions and structure that external contracts can interact with.

- **PrimeStorage.sol**:
  This contract seems to handle storage variables related to the Prime contract. It may be responsible for maintaining crucial data about Prime tokens.

  - **Libraries Used**:
    OracleInterface.sol from Venus Protocol: Likely used to interact with oracles.

- **PrimeLiquidityProvider.sol**:
  This contract provides liquidity to the Prime token in a uniform manner over a specified time period. This liquidity provision is one of the two sources of funds supported by the Prime tokens.

  - **Libraries Used**:
    SafeERC20Upgradeable.sol from OpenZeppelin: For safe handling of ERC20 tokens.
    PausableUpgradeable.sol from OpenZeppelin: For pausing contract operations if needed.
    AccessControlledV8.sol from Venus Protocol: Used for access control.

- **Scores.sol**:
  This is a library contract that contains a function called calculateScore. This function is used to calculate the score associated with a Prime holder, which likely determines the rewards allocated to the user.

  - **Libraries Used**:
    SafeCastUpgradeable.sol from OpenZeppelin: For safe type casting.

- **FixedMath.sol**:
  This is a library contract that likely contains mathematical operations used within the Prime token system.

  - **Libraries Used**:
    SafeCastUpgradeable.sol from OpenZeppelin: For safe type casting.

- **FixedMath0x.sol**:
  This library contract appears to contain additional mathematical operations needed for the Prime token system.

  - **Libraries Used**:
    SafeCastUpgradeable.sol from OpenZeppelin: For safe type casting.

## Approach Taken-in Evaluating The Venus Prime Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Venus Protocol's Prime token system.

    I divided the audit into two main parts. First checked the main contracts I needed to focus on for the contest. Then, I looked at how they connect with other contracts in the Venus Protocole

    **Main Contracts I Looked At**

                Prime.sol
                PrimeLiquidityProvider.sol
                Scores.sol
                FixedMath.sol
                FixedMath0x.sol

    I started my analysis by examining the Prime.sol contract. This contract is the heart of the Prime token system. Its primary purpose is to manage the Prime token and how rewards are earned and claimed by Prime holders. To achieve this, it relies on various libraries and external contracts.

    Then, I turned our attention to the PrimeLiquidityProvider.sol contract. This contract plays a critical role in providing liquidity to the Prime token in a fair and uniform manner over a defined period. This liquidity source is one of the two key methods of funding the Prime tokens. To achieve this, it utilizes the same libraries as the Prime contract (SafeERC20Upgradeable.sol and PausableUpgradeable.sol) and also relies on the AccessControlledV8.sol contract from Venus Protocol for access control.

    And in last examined several library contracts:

    Scores.sol: It contains the essential function 'calculateScore' used to determine the score associated with a Prime holder. This score plays a pivotal role in deciding the rewards allocated to the user.

    FixedMath.sol: Another library, provides fundamental mathematical operations for the Prime token system.

    FixedMath0x.sol: A more extensive library with 181 lines, it provides additional mathematical operations used in the Prime token system.

2.  **Documentation Review**:

    Then went to Review [this document](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/README.md) for a more detailed and technical explanation of Venus Prime.

3.  **Test Coverage Evaluation**: During this phase, I play with the tests, initially encountering challenges when attempting to run them first. However, after resolving the issues, I found the well-written tests to be quite interesting.

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Architecture Description and Diagram

Architecture of the key contracts that are part of the Venus Prime protocol:

**Prime**:- The Prime contract is a critical part of the Venus Prime Protocol. That provides a comprehensive set of functionalities for protocol,including minting and burning Prime tokens, interest accrual on markets, market management with supply and borrow multipliers, score calculation for users, APR and alpha value computation, limits and controls on token minting, pausing and unpausing of certain functions, income distribution, token upgrades, interactions with XVSVault, market addition, asset state updates, and various query functions, enabling users to participate in the protocol's ecosystem and manage their assets effectively.

**PrimeLiquidityProvider**:- The PrimeLiquidityProvider contract is designed for managing the distribution of tokens, with features including initializing token distribution, setting distribution speeds, pausing and resuming fund transfers, sweeping accidental ERC-20 token transfers, and releasing accrued token amounts to a Prime contract, all while utilizing access control and pausing mechanisms.

## Codebase Quality

Overall, I consider the quality of the Venus Prime codebase to be excellent. The code appears to be very mature and well-developed. We have noticed the implementation of various standards, such as ERC20, which Prime.sol and PrimeLiquidityProvider.sol adhere to appropriately. Details are explained below:

| Codebase Quality Categories       | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Clarity and Readability**  | Variable and function names are generally descriptive and follow a consistent naming convention, which helps improve code clarity.For example, variable names like minSavingsRate and minBorrowRate in Prime.sol and tokenDistributionSpeeds and tokenAmountAccrued in PrimeLiquidityProvider.sol are descriptive.Function names like initialize and releaseFunds in PrimeLiquidityProvider.sol convey their intended purposes.                                                                                      |
| **Code Comments**                 | The code includes comments that provide explanations for functions, events, and errors, which can be helpful for understanding the contract's behavior.For instance, comments like @notice and @dev are used to describe the purpose of functions and their parameters.Comments are also used to document events and errors, such as event TokensAccrued and error InvalidArguments.                                                                                                                                 |
| **Documentation**                 | It would be helpful if the docs explained how the ecosystem works from a basic contract level so that it is easier to digest for developers, users and auditors looking to integrate into the Venus Prime Protocol                                                                                                                                                                                                                                                                                                   |
| **Organization**                  | Codebase is very mature and well organized with clear distinctions between the 3 contracts, and their respective interfaces and libraries.                                                                                                                                                                                                                                                                                                                                                                           |
| **Testing**                       | Codebase is well-tested it was great to see the protocol has [testing directory](https://github.com/code-423n4/2023-09-venus/tree/main/tests).                                                                                                                                                                                                                                                                                                                                                                       |
| **Code Structure and Formatting** | The code is logically structured into functions, and there is a clear separation of concerns.Proper indentation and formatting are used, which enhances code readability.Import statements are organized and easy to follow.                                                                                                                                                                                                                                                                                         |
| **Custom Error Types**            | contracts define custom error types using the error keyword.These custom error types are used to throw exceptions in specific error scenarios.Custom error types like InvalidArguments, InvalidDistributionSpeed, InvalidCaller, and others are defined in PrimeLiquidityProvider.sol.Similarly, Prime.sol defines custom error types like InvalidPriceFeed and InsufficientBalance.These custom error types help improve the clarity of error messages and provide more information about why a transaction failed. |
| **Informative Error Messages**    | The contracts provide informative error messages when exceptions are thrown. For example, when a FundsTransferIsPaused error is thrown in PrimeLiquidityProvider.sol, the error message explains that funds transfer is paused.Custom error messages, like "Token already initialized" or "Invalid distribution speed," are used to provide specific details about the error condition.                                                                                                                              |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Venus Prime protocol. These risks encompass concentration risk in Prime, liquidity and yield risk, third-party dependency risk, and centralization risks arising from the existence of an “owner” role in specific contracts. However, the documentation lacks clarity on whether this address represents an externally owned account (EOA) or a contract, warranting the need for clarification. Additionally, the absence of fuzzing and invariant tests could also pose risks to the protocol’s security.

1. **Concentration risk in Prime**:

   If the Prime contract accumulates a significant amount of users’ assets, it could create a systemic risk, especially if the assets are mismanaged or suffer significant losses. Diversifying assets across different platforms and contracts can help mitigate this risk.

2. **Token Distribution Speed**:

   The contract PrimeLiquidityProvider allows the owner to set the distribution speed for different tokens. If the distribution speeds are not carefully managed or if they are set too high, it could lead to a systemic risk. For instance, a high distribution speed might result in excessive token rewards that could destabilize the protocol.

3. - **Token Sweeping**:

   The contract PrimeLiquidityProvider allows the owner to sweep tokens accidentally sent to the contract address. This feature could introduce systemic risks if not implemented securely. Vulnerabilities in the token sweeping process could lead to a loss of funds.

4. **Control Over Alpha Parameters**:

   The calculateScore function takes alphaNumerator and alphaDenominator as parameters, which affect the score calculation. These parameters are controlled by external callers. If these parameters are not set or managed properly, it could introduce systemic risks by affecting the accuracy of score calculations.

5. **Owner Role and Centralization Risk**:

   The contract Prime and PrimeLiquidityProvider has an owner role that can potentially exercise significant control over the contract's functionality.

6. **Market Additions**:

   The contract Prime allows the addition of markets to a prime program. The addition of unsupported or malicious markets could lead to risks if not properly vetted.

7. **Weird ERC-20 Tokens**:

   - If a user deposits malicious or “weird” ERC20 tokens into the Venus Prime protocol, there could be several negative consequences, such as:

     - **Loss of funds**: If the contract is not designed to handle these malicious tokens properly, there could be vulnerabilities that allow an attacker to steal the deposited funds or negatively impact the integrity of the protocol.

     - **Price manipulation**: Malicious tokens could have adverse effects on liquidity and the overall functioning of the protocol, which could affect prices and the stability of the assets involved.

8. **No having, fuzzing and invariant tests could open the door to future vulnerabilities**.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Venus Prime protocol.**

## Gas Optimization

Venus Prime demonstrates a strong commitment to gas optimization, incorporating many widely accepted techniques. In addition to the major optimizations mentioned automatic finding, I have compiled a list of **28** gas optimization suggestions.

To improve code clarity in the provided code snippets, we have occasionally used function abbreviations to highlight specific sections. It's important for developers to exercise caution when integrating these proposed changes to prevent potential vulnerabilities. Even though we have conducted prior testing on these optimizations, developers should take on the responsibility of conducting a thorough reevaluation.

- **Optimizations Related to Arithmetic and Control Flow**:

  - Use ++i instead of i++ to increment

  - Use do-while loops instead of for loops

  - Ternary operation is cheaper than if-else statement

  - Short-circuit booleans

  - Unnecessary casting as the variable is already of the same type

  - Divisions can be unchecked to save gas from previous checks

- **Optimizations Related to Data Structures**:

  - Using Storage instead of memory for structs/arrays saves gas

  - Using mappings instead of arrays to avoid length checks

  - Mappings used within a function more than once should be cached to save gas

- **Optimizations Related to Solidity Features**:

  - Admin functions can be payable

  - Always use Named Returns

  - block.number directly call instead of a function

  - Use hardcoded address instead of address(this)

  - Initializers can be marked as payable to save deployment gas

- **Optimizations Related to Error Handling**:

  - Revert() statements should be used sorted from cheapest to most expensive

  - Using delete statement can save gas

  - Using assembly to revert with an error message

  - Split revert statements

- **Optimizations Related to Code Organization**:

  - Make the variable outside the loop to save gas

  - Should use arguments instead of the state variables

  - Do not calculate constants

  - Shorten the array rather than copying to a new one

- **Optimizations Related to External Libraries**:

  - Use v4.9.0 OpenZeppelin contracts

  - Consider using alternatives to OpenZeppelin

- **Optimizations Related to Event Handling**:

  - Make event parameters indexed when possible

## Conclusion

In general, the Venus Prime project exhibits an interesting and well-developed architecture we believe the team has done a good job
regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from
potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance
understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security
measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project,
Thanks for this beautiful Project.


### Time spent:
15 hours