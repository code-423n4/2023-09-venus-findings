# Analysis of the Venus Prime Codebase

## Approach

The auditing process began with a high-level yet brief overview of the contracts in the ecosystem's scope. Due to the amount of integration from `VToken.sol`, time was also devoted to this. This was followed by examining findings from prior audits available to the public, i.e., audits from FairyProof and PeckShield. Subsequently, a meticulous, line-by-line security review of the entire 1038 sLOC was conducted. I prefer examining the contracts from the ground up, so my approach began with the contracts used as libraries, i.e., FixedMath0x.sol, FixedMath.sol, and Scores.sol. Then I delved into the Prime folder, which contains the three main contracts: PrimeStorage.sol, PrimeLiquidityProvider.sol, & Prime.sol. The final phase entailed examining interactive sessions other security researchers had with the developers on the public discord chat for the contest, enhancing the understanding of unique implementations.

## Architecture Feedback/Risks

- The self-sustainability of Venus Prime is commendable. Users integrating with the system won't need to fret about external factors when considering their rewards.
- Using a minimum duration to increase user loyalty with the protocol is praiseworthy. However, the 90-day duration might seem lengthy to some users.
- Extensive use of factories adds an extra layer of complexity, even if contract sources are from renowned entities like OpenZeppelin.
- Although the code documentation is currently at an acceptable level, there's always room for improvement.
- Venus plans to provide a view function allowing the Venus UI to display an estimated APR related to the Prime token and a user's borrow/supply position. This is commendable. However, it's essential to note that this implementation uses outdated data from `VToken.sol` when calculating the APR in Prime.sol, this is cause the stored values of the exchange rate and borrow balance are used for these calculations instead of their current values
  this uses stale data from `VToken.sol` while calculating the APR in Prime.sol

- Current implementation of calculating staked time duration does not take into account the fact blocks are not mined per seconds on chains where projects would be deployed and as such this could lead to users having to wait for way more time before they can claim their rewards reaching over a thousand days on the Ethereum mainnet.

## Testability

The system's testability is impressive, heavily utilizing Hardhat. Achieving 96% coverage is notable. However, some tests are so lengthy they are challenging to follow. This isn't necessarily a flaw; once familiar with the setup, creating tests and PoCs becomes smoother. The provided high-quality testing sandbox massively aided my efforts in creating a POC that illustrated how users would need to wait approximately three years on the Ethereum mainnet before being able to claim their tokens

## Centralization Risks

- Maintaining trust assumptions is crucial for the project's longevity.
- The `releaseFunds()` function presents a centralization risk. If a malicious admin pauses transfers, all funds are effectively trapped in the contract, severely impacting the `prime.sol` contract. Only it is supposed to access funds from the `PrimeLiquidityProvider.sol` contract.
- Still on the `PrimeLiquidityProvider.sol` contract, I'd like to note that unlike the previous issue which stems from the funds transfer being put in a paused state there is an even bigger issue if the owner ever gets compromised, which is the `sweepToken()` function, do note that an admin can just specify any token's address and have the balance transferred only requirement is that the contract has at least the requested amount of tokens or less, do note that this also massively affects the functionality of the `Prime.sol` contract since an attempt to call on `releaseFunds()` would revert when the `PrimeLiquidityProvider.sol` does not have enough tokens, this could still affect other issues, like accrual of tokens, since the function queries the `balancediff` and in the case where the affected token has been withdrawn or some of it have been withdrawn then `accrueTokens()` would not work since this line would cause a revert ` uint256 balanceDiff = balance - tokenAmountAccrued[token_]`

## Other Recommendations

### **Improve Testability**

As already stated in this report, whereas 96% is a very good amount of coverage since it provides security researchers to build on the already available test suite, it still could be improved, more test cases could be thought through, an idea for a section to improve tests is to ensure that test cases are made for each chain that Prime is going to be deployed on, as each chain is distinct in one way, doing this could have also ensured that the issue with users having to wait for longer durations than their expected stake time would have been correctly mitigated since project is going to see in somewhat of a real time how different blocks time would affect protocol.

### **Enhance Event Monitoring**

Present implementation subtly suggests that event tracking isn't maximized. Instances where events are missing or seem arbitrarily embedded have been observed. A shift towards a more purposeful utilization of events is recommended.

### **Employ More Sanity Checks**

- More sanity checks should be used.

- An instance of this is whenever a subtraaction is being done in protocol, for example as stated in the centralization risks, this line ` uint256 balanceDiff = balance - tokenAmountAccrued[token_]` would cause the the whole accrual of tokens to be bricked, and they are valid steps that could be taken to make this the case, so before directly attempting to make the subtraction a check should first be made to ensure `balance > tokenAmountAccrued[token_]` and if that's not the case then this could be rightly mitigated by either only using the available balance for current accrual or what not
  > NB The just explained scenario is just one of such instances where sanity checks could come in handy, another case could be made for the `accrueTokens()` and more

### **Leverage Additional Auditing Tools**

Many security researchers use Visual Studio Code with specific plugins, such as the [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker). Typos can be indicative of more significant code issues, just as [canaries are to a coal mine](https://en.wiktionary.org/wiki/canary_in_a_coal_mine).

### **Adopt Clearer Naming Conventions**

It's vital to utilize descriptive and intuitive variable names to enhance code readability and documentation. A case in point is the `lastAccruedBlock` mapping. Its current description suggests it represents the _rate_ at which a token is distributed to the prime contract. However, this is misleading, as the mapping in real sense denotes the last block in which an accrual was made for a given address.

### **Add More Developers**

This is intentionally added as the last recommendation, since team is actually packed to a point and this can be seen from the availability of the team members during the duration of the contest, but protocol would always do better with more eyes.

## Security Researcher Logistics

My review of the Venus Prime Codebase took 25 hours spread over three days:

- 1.5 hours for writing this analysis.
- 3 hours reviewing prior audits, available [here](https://github.com/code-423n4/2023-09-venus/tree/main/audits).
- 3 hours (an hour each day) were spent reviewing interactions developers had with other security researchers on the discord group.
- The rest was spent discovering issues and immediately writing reports for them, with some edits made later based on a deeper understanding of the entire protocol.

## Conclusion

The codebase was a very great learning experience, though it was a pretty hard nut to crack, being that it's like an update contest, _codebase has undergone 4 audits_ so it's safe to say that most _easy to catch_ bugs have already been spotted and mitigated.

During the course of my security review, I uncovered a very interesting _Critical/High_ severity issue related to how users would be forced to wait for a very long time before they can claim their prime tokens, using the Ethereum mainnet as a case study, users would be forced to around **3 years** before they can claim their tokens instead of 90 days as currently claimed by Venus.

Additionally, I reported a few other issues I deemed noteworthy, this includes different issues with the `accrueTokens()` function and how `calculateAPR()` does not function properly since it queries stale data from `VToken.sol`. Not to forget the QA Report that was also submitted which entailed a compilation of different QA issues on how to make the Venus Prime codebase better.


### Time spent:
22 hours