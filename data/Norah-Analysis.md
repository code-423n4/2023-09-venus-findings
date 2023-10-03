## Mechanism Overview
Venus Prime implements the reward system for users in order promote the overall protocol engagement. It encourages users to stake substantial amount (1000 xvs) of xvs tokens for longer period of time (minimum 90 days) to qualify for the prime reward program and it also encouraging users to supply and borrow more to be qualified for higher share of the reward. To achieve this protocol implements cobb-douglass formula based reward distribution functionality.

## Recommendation 
While the cobb-douglass formula for reward distribution along with the cap on XVS and user capital is safe choice, it can be still exploited if the protocol reaches a stage wherein qualify user can supply to certain market(get their score updated), then call claimInterest() and finally call withdraws() in same transaction and if the reward earned justifies fees collected by protocol along with gas expenditure, the above malicious sequence can be used to drain all the reward with help of MEV Bots and flashloans.

## Codebase Quality
While the codebase is very clean and thoroughly explained wherever required, there are someplace where dual nomenclature is used like User-Account,Market-vToken etc, which seemed unnecessary.
Apart from there is lack of functionality to update the `comptroller` and `ProtocolShareReserve` addresses along with no routine to remove the `market` added to `markets` array.
Also, please provide basic tests written in foundry as well.

## Approach
Spent total 4 hours a day for 4 days on the contest. 1st day to read the docs and get the feel of codebase, rest for finding the vulnerabilities and write up.






### Time spent:
16 hours