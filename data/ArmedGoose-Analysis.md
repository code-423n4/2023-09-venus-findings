## What is prime?
XVS Prime is a loyalty programme which rewards of are dependent on participating in both XVS staking and borrowing/lending in venus protocol.
The proof of being a member of the Prime program can be minted both by the protocol governance to certain users, or by users themselves if they meet the criteria.
Proof of membership is under the hood increased balance in the Prime.sol contract itself. The contract is called a soulbound token, but it is not implementing any official standard, instead it is a custom contract, however it does not change anything because it is functional as it is.
The entry point function for minting/revoking tokens is function xvsUpdated which takes user address as a parameter and depending on user’s situation issues or burns user tokens, among other options.

## Rewarding
The most important part about Prime is ability to earn rewards for being prime holder. This is the core of the whole idea.
The rewards from this program are accrued per each market added to `Prime.sol` and initialized in `PrimeLiquidityProvider.sol`, they are accrued over time and are based on score calculated on Cobb-Douglas Production function described in the official documentation for the protocol. 

Under the hood, for each market there are following components of the score calculation:
- User’s individual contribution to staking, borrowing and lending and
- Percentage of user’s score share in all total contribution score

The system works in as I called them “micro cycles” as with each update of the score components (balances, capital, xvs, burn, mint) the rewards for past micro cycle (from last accrue timestamp to now) are accrued and accounted, and then the system starts new microcycle from current timestamp (index) to another change. This means, special attention should be paid to intervals of updating, if always new rewards are not accounted to past period for example, IF new changes of components are not introduced before accounting previous system state for past interval.

Aside of that, the system is somewhat centralized from technical point of view (there are roles that control vital aspects of the protocol) however as specified in the documentation it is managed by a governance.
The “vital aspects” are:
- Alpha numerator and denominator being integral part of the score calculation formula
- Adding new markets to participate in Prime program
- Ability to issue OG (irrevocable) tokens, or burn prom tokens

## Threat modeling 
Aside of potential centralization risk, which is greatly reduced by using governance, the code has been reviewed and suitable vulnerabilities (if any found) were submitted. The threat modeling shows following areas that could be interesting for the protocol:
- Can the user manipulate the Prime tokens, have multiple tokens, or influence Prime tokens balance in other malicious way?
- Can user manipulate the rewards amount? Can user DoS other users?
- Is the formula and the reward calculation implemented correctly?
- Can user manipulate the eligibility criteria?
- Are all entries to the system properly updating Prime calculation components properly?
- Is it possible to put protocol in an inconsistent state, where it will stop working partially or entirely?

## Overall summary
Overall the code quality is good and consistent. Several possible concerns were raised as separate submissions, including: possibility of accidentally putting the system at inconsistent state, possible wrong reward calculation or possible concerns related multiple loops over storage.




### Time spent:
40 hours