# Approach taken in evaluating the codebase


During this audit, my main focus was on examining the rewards and staking system in the Venus Prime protocol.

Day 1: I spent time understanding the overall working of the Venus Prime protocol and getting an overview of the codebase.

Day 2: I conducted a detailed exploration of the rewards and staking mechanisms. 

Day 3: I identified potential gas optimization issues that could be addressed by protocol team to save gas.

Day 4: I dedicated this day to preparing the final report and analysis, summarizing the findings and recommendations.



# Architecture recommendations




## Consensus Mechanism: 

Consider implementing a mechanism for consensus among a group of admins to reduce centralization risk

## Gas Optimizations

Review data types : Analyze the data types used in smart contracts and consider if they can be further optimized. For example, changing uint256 to uint128 or uint94 can save gas and storage slots. 
Struct packing : Look for opportunities to pack structs into fewer storage slots. By carefully selecting appropriate data types for struct members, this can reduce the overall storage usage. 
Use constant values : If certain values in contracts are constant and do not change, declare them as constants rather than storing them as state variables. This can significantly save gas costs. 
Avoid unnecessary storage : Examine your code and eliminate any unnecessary storage of variables or addresses that are not required for contract functionality. 


## Other recommendations

Regular code reviews and adherence to best practices. 
Conduct external audits by security experts. 
Consider open sourcing the contract for community review. 
Maintain comprehensive security documentation. 
Establish a responsible disclosure policy for vulnerabilities. 
Implement continuous monitoring for unusual activity. 
Educate users about risks and best practices.




# Codebase quality analysis

##Prime.sol

The contract does not have explicit access control modifiers, such as onlyOwner or onlyAuthorized , to restrict access to sensitive functions.
Code does not follow the best practice of check-effects-interaction
Missing checks for address(0) in initializer
Initialize function lacks parameter validation
External calls in an un-bounded for -loop may result in a DOS

##PrimeLiquidityProvider.sol

The owner is a single point of failure and a centralization risk.Consider changing to a multi-signature setup, or having a role-based authorization model.
Missing checks for address(0) in initializer 
Initialization can be front-run Since 2 initialize() functions are not called by another contract atomically after the contract is deployed.
Large transfers may not work with some ERC20 tokens. Some IERC20 implementations (e.g UNI , COMP ) may fail if the valued transferred is larger than uint96.
Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions

##PrimeStorage.sol

State variables should include comments. EXP_SCALE state variable lacks comments.
PrimeStorageV1 contract which contains only utility functions should be made into libraries
The following constants doesn't use scientific notation for readability: MINIMUM_STAKED_XVS, MAXIMUM_XVS_CAP, and MAXIMUM_BPS.

##Scores.sol

The “calculateScore” internal function which is not called by the contract should be removed to save deployment gas.
NatSpec: Contract declarations lack @title annotations 
Contract does not Use the latest solidity version for deployment

##FixedMath.sol

Contract does not use --via-ir that activates more advanced multi-function optimizations, for extra gas savings
Contract custom error has no error details. If parameters are added to errors, it will indicate which user or values caused the failure

##FixedMath0x.sol

The contract lacks detailed inline comments explaining the purpose and functionality of the code.
Contract uses long functions e.g “in()”, and “exp()” that should be refactored into multiple, smaller, functions 

##IPrime.sol

Contract does not Use the latest solidity version for deployment




# Centralization risks

The protocol has made significant progress towards decentralization from Venus (Original) to Venus Prime, and the efforts to achieve this are commendable.

Having a single EOA as the only owner of contracts is a large centralization risk and a single point of failure. A single private key may be taken in a hack, or the sole holder of the key may become unable to retrieve the key when necessary. Consider changing to a multi-signature setup, or having a role-based authorization model. Centrality risk is high in the project as the role of onlyOwner detailed below has very critical and important powers:

File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol 

118 function initializeTokens(address[] calldata tokens_) external onlyOwner { 

177 function setPrimeToken(address prime_) external onlyOwner { 

216 function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {




# Mechanism review

The mechanisms implemented in the Venus Prime protocol, including incentive program aimed to bolster user engagement and growth are revolutionary indeed. Venus Prime's uniqueness lies in its self-sustaining rewards system, instead of external sources, rewards are derived from the protocol's revenue, fostering a sustainable and ever-growing program.

Non the less, there are some potential issues and risks associated with these mechanisms. For example, the function _calculateUserAPR() does not update the oracle for the input market or xvsToken. Thus the oracle will not update the pivot oracle price before calculating a users capital for their score. This issues should be carefully considered and mitigated to ensure the security and reliability of the protocol.




# Systemic risks


Dependency Risk: Venus Prime heavily depends on Openzepplins’s contracts. If there are bugs or vulnerabilities in those contracts, or if they change their behavior, it could impact the functioning of Venus. 

Centralization: Centralizing control in the prime.sol contract can be a systemic risk if the contract has bugs or is compromised

Like any smart contract-based system, Venus Prime is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol.

Test Coverage: the test coverage provided by Venus Prime is 96%, however I recommend 100% of the test coverage

### Time spent:
32 hours