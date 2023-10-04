## Approach

Days before the contest started I researched information on upcoming changes on Venus protocol. The v4 whitepaper was found and studied (https://github.com/VenusProtocol/venus-protocol-documentation/blob/main/whitepapers/Venus-whitepaper-v4.pdf). This served me well as the context and terminology were already clear to me once the context started which should have given me a head start.

I had already participated in the previous Venus contest. I revisited my old contest notes and the published audit report to have an idea about potential attack vectors developers did not cover in the previous contest.

As soon as the technical documentation (https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/README.md) was accessible I read it thoroughly.

Once I had some context knowledge, I went through all the contracts (sorted by ascending SLOC) executing a manual review and noting finings and potential attack vectors using @audit tags.

For FixedMath0x.sol I downloaded the original unmodified file, did a diff with VS Code and reviewed the diff manually.

I drew a diagram consisting of Prime.sol, XVSVault.sol, PolicyFacet.sol and PrimeLiquidityProvider.sol mapping out the publicly callable functions and their interactions.

In this contest I found the token issuing/upgrading/burning logic quite interesting and put a higher focus on it to find edge cases as I thought the pure reward calculation math may have been probed by the Venus team with focus. I also thought that competing auditors would focus on the math which would not give me an edge if I focused here as well.

I manually validated whether all functions that have "_checkAccessAllowed" as their first call have the correct function signature set so access control works.

If I found leads considered solid I covered them with POCs (either directly coded or manually calculated). I cleaned these POCs up and included them in my submitted issues.

## Observations and obstacles during the review

### Test suite

I am more familiar with Foundry and less with Hardhat. That cost me some valuable time writing POCs. It was good training though to get more accustomed to Hardhat and JS.

### Outdated / missing NatSpec

I once went down a rabbit hole writing a POC for more than 1h to validate whether _executeBoost() of Prime.sol "Must be called by Comptroller before changing account's borrow or supply balance." was folowed in the implementation. Once I knew it was not called before I validated my finding with the Venus protocol team via Discord. Turned out the NatSpec was outdated.

I would have wished for NatSpec in interface as well. It is e.g. missing for IPrime.sol. This would have given me a better option to get a high-level overview of a contract's functionality quicker without the visual distraction of the function implementations. I recommend to add this NatSpec.

Other than that the NatSpec seems solid apart from a few typos here and there.

### Missing interaction diagram

It would have been beneficial if the Venus protocol team had provided a diagram that is more than only a high-level overview. Once that shows relations and interactions between contracts more in-depth. I had to draw my own by reverse engineering the contract interactions.

## Codebase Quality
| Category         | Feedback                                                                                                                                                                                                                                                                                                                    |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Documentation    | Documentation was overall good. More diagrams and NatSpec in contract interfaces would have been beneficial.                                                                                                                                                                                                                |
| Code Comments    | Code comments overall were sparse. I had to go through many functions, break them apart, and add my own inline comments to make more sense of them.                                                                                                                                                                           |
| Code Syntax      | Code Syntax was good. I guess some automated formatting was applied. Some inconsistencies exist, e.g., when setting values to 0 or incrementing a loop index.                                                                                                                                                                 |
| Organization     | The codebase was compact and well-organized. No issue here.                                                                                                                                                                                                                                                                |
| Unit Testing     | The codebase had a fully functional Hardhat suite that had extensive unit tests as well as integration tests. A command was provided to only run the tests related to Prime. Fuzz-testing could not be found. Fuzzing may be easier to integrate if the test suite was ported to Foundry. Adding fuzz-testing critical functions (e.g. reward calculations) is highly encouraged. |

## Systemic/Centralization Risks

- The protocol has many functions that are protected by _checkAccessAllowed calls
- The creation/destruction of Prime tokens is not fully permissionless since admin controlled functions exist to issue and burn the token of every user.
- Since the protocol will also be deployed on L2s (Arbitrum, Polygon zkEVM, opBNB) where the cost of stuffing a block can be as low as $1, block stuffing attacks are a very valid risk which even can cost users rewards. This already led to findings in the last Venus contest on Code4rena and needs to be carefully reviewed.

## Learnings

- As I had in parts a focus on the issuing/upgrading/burning logic of Prime tokens and its side-effects I found the concept of "upgrading" tokens interesting. I also learned that having a permissionless token claiming/burning implementation alongside an admin-controlled permissioned token issue/burning system is hard to get right as there may be edge cases.

- I also liked the way how tokens are accrued from lending/borrowing protocol operations as well as additionally from the incentive program of the PrimaryLiquidityProvider. The flow of supplying distributable tokens, accruing them and claiming them is interesting. Also the concept of the reward index and score was joyful to learn.

### Time spent: 36 hours

### Time spent:
36 hours