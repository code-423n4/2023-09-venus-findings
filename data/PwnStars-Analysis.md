# Our Analysis Of The Codebase


Me with my team mate qb decided to take part in the Venus Prime contest on Code4rena with the goal of finding as many flaws in the system and help the protocol to be as secure as possible before it’s launch.
The codebase (the scope for the contest) was focused mainly on two contracts i.e. PrimeLiquidityProvider.sol and Prime.sol . To explain the system in simple terms a user needs to stake $XVS to receive a prime token which is used to boost rewards across markets . The user needs to stake 1000 XVS minimum for at least 90 days in order to be eligible for the prime token.


## Staking And Liquidation

If the user has staked minimum required XVS he can (or someone else can) set his stakedAt[] mapping to the current timestamp and then later the person can call claim() to claim his prime token , if the user unstakes before 90 days then his stake is subject to liquidation where any other user can call xvsUpdated and reset the the stakedAt[] mapping back to 0 making the user unable to claim the prime token.

## Revocable Versus Irrevocable Prime Tokens

There are two categories for prime tokens , revocable and irrevocable . Under normal conditions the user would be minted a revocable Prime token if he fulfills the criteria , a revocable token can be burnt (via xvsUpdated ) if the XVS staked by the user falls below 1000.

An irrevocable prime token is more powerful , it is independent of stakes and can not be burnt . The only way a prime token is issued to a user is through the issue() function where a privileged actor can mint a user an irrevocable token which accrues interest and enjoys market rewards.
A pause mechanism is also implemented during which a user can not claim boosted yield.

# Our Approach And Advice

Our approach was plain and simple , have an initial overview of the system and understand each component → draw out a graph to understand the communications between different parts of the code → clear out any doubts from the sponsor → finally diving into the codebase and attacking the critical logic first and diving deep into possibilities and edge cases.

![Graph1](https://user-images.githubusercontent.com/93149832/272659098-91398730-b679-49cb-8c38-03284f415eb8.png)
![Graph2](https://user-images.githubusercontent.com/93149832/272661339-3be88565-d515-4100-94ea-9a709ade603b.png)
![Graph3](https://user-images.githubusercontent.com/93149832/272661482-ea73925f-2171-4d4c-af9e-6613c6cdb49d.png)

# Broader Concerns

First thing we noticed was that the system is prone to centralisation risks . The owner holds the privilege to set the address of the prime token which is the reward token , the owner can sweep any token from the contract directly to his account . But upon reading documentation we realised the team has already acknowledged this and use timelocked contracts.

# Methodology/Approach

We decided to deep dive into the mintng and liquidation process to find flaws and edge cases in the logic. Eventhough we managed to find a couple of good edge cases , we concluded that code is robust and was written with security as the main priority. Our approach here was to break the logic into all possible states and possibilities and look through each and conclude if it could have been problematic. For example - what if user already has staked XVS and he gets a prime token issued by the owner? Is there any possibility which lets a user stake then unstake in a series to claim a prime token?


The next section was dedicated to external libraries used or any fork.
Our observation and approach → • In PolicyFacet.sol, after executing any operation that could impact the Prime score or interest, we accrue the interest and update the score for the prime user by calling accrueInterestAndUpdateScore.


The vToken is forked from compound , we raised questions like is there a way to manipulate capital value (example - using flashloans) and accrue huge amounts of interest leading to drainage of assets? A thorough analysis was done to asses all the risks that could have arose with compound integration.


After applying the above methodologies which were somewhat formal we started to find for flaws through a miscellaneous approach . We dissected the system into different parts and flows and verified each . This included testing to see if a variable were not updated where it should have been leading to inconsistencies , verifying if the interest was accrued everytime when the reward index was read , the scores of a user and total scores were updated correctly and checking if accrued interest was distributed fairly and if there was a way the rewards could have been lost.


# New insights and learnings from the audit

We went above and beyond to find flaws in the Venus Prime system , since the codebase was already heavily audited it was tough to find anything . We had around 5 rounds of code review where we fought hard to find anything which we could categorize as a bug and went deep inside the system.

It was because of this “keep going” attitude we were successful in finding bugs in such a heavily audited codebase . We learnt that an auditor has to believe that there still exists bugs in the system regardless of how bulletproof the system might be.

Venus Prime used Cobb-Douglas function in order to calculate score and rewards . Efforts were made to understand the core logic of this function and how it applies to the reward system and look at reward distribution through a mathematical point of view.

To make things clearer throughout the process we built flow graphs and made interactive notes to make sense of the state changes that happened throughout . This proved out to be essential in understanding the system better and make the whole audit process effecient and better.



### Time spent:
30 hours