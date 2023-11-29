---
sponsor: "Venus Protocol"
slug: "2023-09-venus"
date: "2023-11-29"
title: "Venus Prime"
findings: "https://github.com/code-423n4/2023-09-venus-findings/issues"
contest: 290
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Venus Prime smart contract system written in Solidity. The audit took place between September 28—October 4 2023.

## Wardens

130 Wardens contributed reports to Venus Prime:

  1. [Testerbot](https://code4rena.com/@Testerbot)
  2. [0xTheC0der](https://code4rena.com/@0xTheC0der)
  3. [0xDetermination](https://code4rena.com/@0xDetermination)
  4. [Brenzee](https://code4rena.com/@Brenzee)
  5. [ether\_sky](https://code4rena.com/@ether_sky)
  6. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  7. [Breeje](https://code4rena.com/@Breeje)
  8. [tapir](https://code4rena.com/@tapir)
  9. [santipu\_](https://code4rena.com/@santipu_)
  10. [sces60107](https://code4rena.com/@sces60107)
  11. [ast3ros](https://code4rena.com/@ast3ros)
  12. [pep7siup](https://code4rena.com/@pep7siup)
  13. [ThreeSigma](https://code4rena.com/@ThreeSigma) ([0x73696d616f](https://code4rena.com/@0x73696d616f), [0xCarolina](https://code4rena.com/@0xCarolina), [EduCatarino](https://code4rena.com/@EduCatarino), and [SolidityDev99](https://code4rena.com/@SolidityDev99))
  14. [said](https://code4rena.com/@said)
  15. [3agle](https://code4rena.com/@3agle)
  16. [rokinot](https://code4rena.com/@rokinot)
  17. [PwnStars](https://code4rena.com/@PwnStars) ([qbs](https://code4rena.com/@qbs) and [sakshamguruji](https://code4rena.com/@sakshamguruji))
  18. [neumo](https://code4rena.com/@neumo)
  19. [blutorque](https://code4rena.com/@blutorque)
  20. [Pessimistic](https://code4rena.com/@Pessimistic) ([olegggatttor](https://code4rena.com/@olegggatttor), [yhtyyar](https://code4rena.com/@yhtyyar), and [PavelCore](https://code4rena.com/@PavelCore))
  21. [DavidGiladi](https://code4rena.com/@DavidGiladi)
  22. [oakcobalt](https://code4rena.com/@oakcobalt)
  23. [hals](https://code4rena.com/@hals)
  24. [bin2chen](https://code4rena.com/@bin2chen)
  25. [DeFiHackLabs](https://code4rena.com/@DeFiHackLabs) ([AkshaySrivastav](https://code4rena.com/@AkshaySrivastav), [Cache\_and\_Burn](https://code4rena.com/@Cache_and_Burn), [IceBear](https://code4rena.com/@IceBear), [Ronin](https://code4rena.com/@Ronin), [Sm4rty](https://code4rena.com/@Sm4rty), [SunSec](https://code4rena.com/@SunSec), [sashik\_eth](https://code4rena.com/@sashik_eth), [zuhaibmohd](https://code4rena.com/@zuhaibmohd), and [ret2basic](https://code4rena.com/@ret2basic))
  26. [Norah](https://code4rena.com/@Norah)
  27. [seerether](https://code4rena.com/@seerether)
  28. [turvy\_fuzz](https://code4rena.com/@turvy_fuzz)
  29. [dirk\_y](https://code4rena.com/@dirk_y)
  30. [deadrxsezzz](https://code4rena.com/@deadrxsezzz)
  31. [0xprinc](https://code4rena.com/@0xprinc)
  32. [0x3b](https://code4rena.com/@0x3b)
  33. [deth](https://code4rena.com/@deth)
  34. [J4X](https://code4rena.com/@J4X)
  35. [Satyam\_Sharma](https://code4rena.com/@Satyam_Sharma)
  36. [gkrastenov](https://code4rena.com/@gkrastenov)
  37. [merlin](https://code4rena.com/@merlin)
  38. [Flora](https://code4rena.com/@Flora)
  39. [KrisApostolov](https://code4rena.com/@KrisApostolov)
  40. [berlin-101](https://code4rena.com/@berlin-101)
  41. [twicek](https://code4rena.com/@twicek)
  42. [0xpiken](https://code4rena.com/@0xpiken)
  43. [rvierdiiev](https://code4rena.com/@rvierdiiev)
  44. [HChang26](https://code4rena.com/@HChang26)
  45. [mahdirostami](https://code4rena.com/@mahdirostami)
  46. [lsaudit](https://code4rena.com/@lsaudit)
  47. [aycozynfada](https://code4rena.com/@aycozynfada)
  48. [sl1](https://code4rena.com/@sl1)
  49. [0xhacksmithh](https://code4rena.com/@0xhacksmithh)
  50. [pavankv](https://code4rena.com/@pavankv)
  51. [Bauchibred](https://code4rena.com/@Bauchibred)
  52. [maanas](https://code4rena.com/@maanas)
  53. [josephdara](https://code4rena.com/@josephdara)
  54. [0xweb3boy](https://code4rena.com/@0xweb3boy)
  55. [al88nsk](https://code4rena.com/@al88nsk)
  56. [xAriextz](https://code4rena.com/@xAriextz)
  57. [pina](https://code4rena.com/@pina)
  58. [SBSecurity](https://code4rena.com/@SBSecurity) ([Slavcheww](https://code4rena.com/@Slavcheww) and [Blckhv](https://code4rena.com/@Blckhv))
  59. [dethera](https://code4rena.com/@dethera)
  60. [0xblackskull](https://code4rena.com/@0xblackskull)
  61. [tsvetanovv](https://code4rena.com/@tsvetanovv)
  62. [ADM](https://code4rena.com/@ADM)
  63. [debo](https://code4rena.com/@debo)
  64. [ArmedGoose](https://code4rena.com/@ArmedGoose)
  65. [jkoppel](https://code4rena.com/@jkoppel)
  66. [0xWaitress](https://code4rena.com/@0xWaitress)
  67. [radev\_sw](https://code4rena.com/@radev_sw)
  68. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  69. [kaveyjoe](https://code4rena.com/@kaveyjoe)
  70. [versiyonbir](https://code4rena.com/@versiyonbir)
  71. [jamshed](https://code4rena.com/@jamshed)
  72. [pontifex](https://code4rena.com/@pontifex)
  73. [hihen](https://code4rena.com/@hihen)
  74. [0xScourgedev](https://code4rena.com/@0xScourgedev)
  75. [Maroutis](https://code4rena.com/@Maroutis)
  76. [inzinko](https://code4rena.com/@inzinko)
  77. [ge6a](https://code4rena.com/@ge6a)
  78. [0xMosh](https://code4rena.com/@0xMosh)
  79. [btk](https://code4rena.com/@btk)
  80. [0xdice91](https://code4rena.com/@0xdice91)
  81. [Fulum](https://code4rena.com/@Fulum)
  82. [imare](https://code4rena.com/@imare)
  83. [Aymen0909](https://code4rena.com/@Aymen0909)
  84. [SPYBOY](https://code4rena.com/@SPYBOY)
  85. [jnforja](https://code4rena.com/@jnforja)
  86. [alexweb3](https://code4rena.com/@alexweb3)
  87. [Mirror](https://code4rena.com/@Mirror)
  88. [0xTiwa](https://code4rena.com/@0xTiwa)
  89. [d3e4](https://code4rena.com/@d3e4)
  90. [tonisives](https://code4rena.com/@tonisives)
  91. [MohammedRizwan](https://code4rena.com/@MohammedRizwan)
  92. [Tricko](https://code4rena.com/@Tricko)
  93. [0xfusion](https://code4rena.com/@0xfusion)
  94. [glcanvas](https://code4rena.com/@glcanvas)
  95. [lotux](https://code4rena.com/@lotux)
  96. [y4y](https://code4rena.com/@y4y)
  97. [sashik\_eth](https://code4rena.com/@sashik_eth)
  98. [Krace](https://code4rena.com/@Krace)
  99. [Daniel526](https://code4rena.com/@Daniel526)
  100. [squeaky\_cactus](https://code4rena.com/@squeaky_cactus)
  101. [joaovwfreire](https://code4rena.com/@joaovwfreire)
  102. [peanuts](https://code4rena.com/@peanuts)
  103. [vagrant](https://code4rena.com/@vagrant)
  104. [nobody2018](https://code4rena.com/@nobody2018)
  105. [nisedo](https://code4rena.com/@nisedo)
  106. [IceBear](https://code4rena.com/@IceBear)
  107. [terrancrypt](https://code4rena.com/@terrancrypt)
  108. [Hama](https://code4rena.com/@Hama)
  109. [n1punp](https://code4rena.com/@n1punp)
  110. [nadin](https://code4rena.com/@nadin)
  111. [e0d1n](https://code4rena.com/@e0d1n)
  112. [kutugu](https://code4rena.com/@kutugu)
  113. [TangYuanShen](https://code4rena.com/@TangYuanShen)
  114. [orion](https://code4rena.com/@orion)
  115. [ptsanev](https://code4rena.com/@ptsanev)

This audit was judged by [0xDjango](https://code4rena.com/@0xdjango).

Final report assembled by [PaperParachute](https://twitter.com/PaperP_C4).

# Summary

The C4 analysis yielded an aggregated total of 5 unique vulnerabilities. Of these vulnerabilities, 3 received a risk rating in the category of HIGH severity and 3 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 88 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 11 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Venus Prime repository](https://github.com/code-423n4/2023-09-venus), and is composed of 7 smart contracts written in the Solidity programming language and includes 1039 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **Tera Bot** from warden [kn0t](https://code4rena.com/@kn0t), generated the [Automated Findings report](https://gist.github.com/code423n4/9e80eddfb29953d8b5a424084a54e4ed) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (3)
## [[H-01] Prime.sol - User can claim Prime token without having any staked XVS, because his `stakedAt` isn't reset whenever he is issued an irrevocable token](https://github.com/code-423n4/2023-09-venus-findings/issues/633)
*Submitted by [deth](https://github.com/code-423n4/2023-09-venus-findings/issues/633), also found by [rokinot](https://github.com/code-423n4/2023-09-venus-findings/issues/684), [gkrastenov](https://github.com/code-423n4/2023-09-venus-findings/issues/680), [merlin](https://github.com/code-423n4/2023-09-venus-findings/issues/645), [Testerbot](https://github.com/code-423n4/2023-09-venus-findings/issues/592), [0xDetermination](https://github.com/code-423n4/2023-09-venus-findings/issues/553), [3agle](https://github.com/code-423n4/2023-09-venus-findings/issues/469), [aycozynfada](https://github.com/code-423n4/2023-09-venus-findings/issues/468), [Flora](https://github.com/code-423n4/2023-09-venus-findings/issues/447), [KrisApostolov](https://github.com/code-423n4/2023-09-venus-findings/issues/439), [berlin-101](https://github.com/code-423n4/2023-09-venus-findings/issues/273), [santipu\_](https://github.com/code-423n4/2023-09-venus-findings/issues/255), [twicek](https://github.com/code-423n4/2023-09-venus-findings/issues/251), [sl1](https://github.com/code-423n4/2023-09-venus-findings/issues/215), [0xpiken](https://github.com/code-423n4/2023-09-venus-findings/issues/211), [Brenzee](https://github.com/code-423n4/2023-09-venus-findings/issues/199), [rvierdiiev](https://github.com/code-423n4/2023-09-venus-findings/issues/145), [tapir](https://github.com/code-423n4/2023-09-venus-findings/issues/144), [HChang26](https://github.com/code-423n4/2023-09-venus-findings/issues/109), [Satyam\_Sharma](https://github.com/code-423n4/2023-09-venus-findings/issues/106), [mahdirostami](https://github.com/code-423n4/2023-09-venus-findings/issues/102), and [said](https://github.com/code-423n4/2023-09-venus-findings/issues/66)*

Whenever a new Prime token is created, the users `stakedAt` is reset to 0. This happens when the user `claim` a revocable token and when he is `issue` a revocable token, but it does not happen when a user is `issue` an irrevocable token.

This is `issue()`

```javascript
function issue(bool isIrrevocable, address[] calldata users) external {
        _checkAccessAllowed("issue(bool,address[])");

        if (isIrrevocable) {
            for (uint256 i = 0; i < users.length; ) {
                Token storage userToken = tokens[users[i]];
                if (userToken.exists && !userToken.isIrrevocable) {
                    _upgrade(users[i]);
                } else {
                    // We don't reset here.
                    _mint(true, users[i]);
                    _initializeMarkets(users[i]);
                }

                unchecked {
                    i++;
                }
            }
        } else {
            for (uint256 i = 0; i < users.length; ) {
                _mint(false, users[i]);
                _initializeMarkets(users[i]);
                
                // We reset stakedAt here
                delete stakedAt[users[i]];

                unchecked {
                    i++;
                }
            }
        }
    }
```

We can see that when a revocable token is issued and minted the user's `stakedAt` is reset to 0. Whenever a user's token is upgraded, his `stakedAt` has already been reset to 0 inside `claim`.

```javascript
function claim() external {
        if (stakedAt[msg.sender] == 0) revert IneligibleToClaim();
        if (block.timestamp - stakedAt[msg.sender] < STAKING_PERIOD) revert WaitMoreTime();
        
        // We reset stakedAt here
        stakedAt[msg.sender] = 0;

        _mint(false, msg.sender);
        _initializeMarkets(msg.sender);
    }
```

The only one time when we don't reset the user's `stakedAt` and it's when he is issued an irrevocable token.

Let's see an example and see why this is a problem:

1.  Alice deposits 10k XVS.
2.  The protocol/DAO/admin decides to issue Alice an irrevocable prime token, because she deposited such a large amount of tokens. Keep in mind that the 90 day staking period still hasn't passed and her `stakedAt` is the original time that she deposited 10k XVS.
3.  Time passes and Alice decides to withdraw her entire XVS, so now she has 0 XVS. Her token isn't burned as she has an irrevocable token.
4.  Even more time passes and the protocol/DAO/admin decides to burn Alice's irrevocable token because she is inactive.
5.  EVEN more time passes and Alice returns to the protocol and instead of depositing anything, she calls `claim`.

Her tx goes through, since her `stakedAt` wasn't reset to 0 when she got issued her irrevocable token.

This way, Alice claimed a revocable token without having any XVS staked in the contract.

### Proof of Concept

Add the following line at the top of `tests/hardhat/Prime/Prime.ts`. We'll use this to simulate time passing

```javascript
import { time } from "@nomicfoundation/hardhat-network-helpers";
```

Paste the following inside `tests/hardhat/Prime/Prime.ts` and run `npx hardhat test tests/hardhat/Prime/Prime.ts`.

```javascript
it.only("User can get Prime token without any XVS staked", async () => {
      // User1 deposits 10k XVS
      await xvs.transfer(await user1.getAddress(), parseUnits("10000", 18));
      await xvs.connect(user1).approve(xvsVault.address, parseUnits("10000", 18));
      await xvsVault.connect(user1).deposit(xvs.address, 0, parseUnits("10000", 18));
      let userInfo = await xvsVault.getUserInfo(xvs.address, 0, user1.getAddress());
      expect(userInfo.amount).to.eq(parseUnits("10000", 18));

      // Venus decides to issue an irrevocable Prime token to User1 for staking such a large amount.
      // Note that the 90 day staking period still hasn't passed
      await prime.issue(true, [user1.getAddress()]);
      let token = await prime.tokens(user1.getAddress());
      expect(token.exists).to.be.equal(true);
      expect(token.isIrrevocable).to.be.equal(true);

      // User1 withdraws her entire balance XVS
      await xvsVault.connect(user1).requestWithdrawal(xvs.address, 0, parseUnits("10000", 18));
      userInfo = await xvsVault.getUserInfo(xvs.address, 0, user1.getAddress());
      expect(userInfo.pendingWithdrawals).to.eq(parseUnits("10000", 18));

      // User1's Prime token gets burned by protocol
      await prime.burn(user1.getAddress());
      token = await prime.tokens(user1.getAddress());
      expect(token.exists).to.be.equal(false);
      expect(token.isIrrevocable).to.be.equal(false);

      // 100 days pass
      await time.increase(8640000);

      // User1 can claim a revocable Prime token without any XVS staked, because his stakedAt wasn't reset to 0
      expect(prime.stakedAt(await user1.getAddress())).to.not.be.equal(0);

      await prime.connect(user1).claim();
      token = await prime.tokens(user1.getAddress());
      expect(token.exists).to.be.equal(true);
      expect(token.isIrrevocable).to.be.equal(false);
    });
```

If you are having trouble running the test, this change might fix it. Inside `Prime.sol`, `burn()` remove the access control from the function. This doesn't change the attack and the test outcome.

```javascript
function burn(address user) external {
        // _checkAccessAllowed("burn(address)");
        _burn(user);
    }
```

### Tools Used

Hardhat

### Recommended Mitigation Steps

Reset the user's `stakedAt` whenever he is issued an irrevocable token.

```javascript
function issue(bool isIrrevocable, address[] calldata users) external {
        _checkAccessAllowed("issue(bool,address[])");

        if (isIrrevocable) {
            for (uint256 i = 0; i < users.length; ) {
                Token storage userToken = tokens[users[i]];
                if (userToken.exists && !userToken.isIrrevocable) {
                    _upgrade(users[i]);
                } else {
                    _mint(true, users[i]);
                    _initializeMarkets(users[i]);
                    delete stakedAt[users[i]]; 
                }
          ...

```

**[chechu (Venus) confirmed and commented](https://github.com/code-423n4/2023-09-venus-findings/issues/633#issuecomment-1777540916):**
 > Fixed [here](https://github.com/VenusProtocol/venus-protocol/commit/9d318165333b5ae4d2350a4b674d7d7237066642).

**[0xDjango (Judge) commented](https://github.com/code-423n4/2023-09-venus-findings/issues/633#issuecomment-1787932866):**
 > Valid issue with serious impact to the protocol.


***

## [[H-02] A malicious user can avoid unfavorable score updates after alpha/multiplier changes, resulting in accrual of outsized rewards for the attacker at the expense of other users](https://github.com/code-423n4/2023-09-venus-findings/issues/555)
*Submitted by [0xDetermination](https://github.com/code-423n4/2023-09-venus-findings/issues/555), also found by [hals](https://github.com/code-423n4/2023-09-venus-findings/issues/685), [Testerbot](https://github.com/code-423n4/2023-09-venus-findings/issues/586), [bin2chen](https://github.com/code-423n4/2023-09-venus-findings/issues/545), [Pessimistic](https://github.com/code-423n4/2023-09-venus-findings/issues/537), [rokinot](https://github.com/code-423n4/2023-09-venus-findings/issues/533), ThreeSigma ([1](https://github.com/code-423n4/2023-09-venus-findings/issues/530), [2](https://github.com/code-423n4/2023-09-venus-findings/issues/524)), [ether\_sky](https://github.com/code-423n4/2023-09-venus-findings/issues/505), [PwnStars](https://github.com/code-423n4/2023-09-venus-findings/issues/463), [neumo](https://github.com/code-423n4/2023-09-venus-findings/issues/427), [DeFiHackLabs](https://github.com/code-423n4/2023-09-venus-findings/issues/346), [turvy\_fuzz](https://github.com/code-423n4/2023-09-venus-findings/issues/341), [Norah](https://github.com/code-423n4/2023-09-venus-findings/issues/304), [dirk\_y](https://github.com/code-423n4/2023-09-venus-findings/issues/295), [deadrxsezzz](https://github.com/code-423n4/2023-09-venus-findings/issues/231), [blutorque](https://github.com/code-423n4/2023-09-venus-findings/issues/217), [SpicyMeatball](https://github.com/code-423n4/2023-09-venus-findings/issues/136), [seerether](https://github.com/code-423n4/2023-09-venus-findings/issues/104), and [said](https://github.com/code-423n4/2023-09-venus-findings/issues/60)*

<https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L397-L405> 

<https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L704-L756> 

<https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L623-L639> 

<https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L827-L833> 

<https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L219> 

<https://github.com/code-423n4/2023-09-venus/blob/main/tests/hardhat/Prime/Prime.ts#L294-L301>

Please note: All functions/properties referred to are in the `Prime.sol` contract.

### Impact

A malicious user can accrue outsized rewards at the expense of other users after `updateAlpha()` or `updateMultipliers()` is called.

### Proof of Concept

An attacker can prevent their score from being updated and decreased after the protocol's alpha or multipliers change. This is done by manipulatively decreasing the value of `pendingScoreUpdates`, then ensuring that only other user scores are updated until `pendingScoreUpdates` reaches zero, at which point calls to `updateScores()` will revert with the error `NoScoreUpdatesRequired()`. This can be done via the attacker calling `updateScores()` to update other users' scores first and/or DoSing calls to `updateScores()` that would update the attacker's score (see the issue titled "DoS and gas griefing of Prime.updateScores()").

The core of this vulnerability is the attacker's ability to manipulate `pendingScoreUpdates`. Notice below that `claim()`, which is called to mint a user's Prime token, doesn't change the value of `pendingScoreUpdates`:

<details>

        function claim() external { 
            if (stakedAt[msg.sender] == 0) revert IneligibleToClaim();
            if (block.timestamp - stakedAt[msg.sender] < STAKING_PERIOD) revert WaitMoreTime();

            stakedAt[msg.sender] = 0;

            _mint(false, msg.sender);
            _initializeMarkets(msg.sender);
        }
        function _mint(bool isIrrevocable, address user) internal {
            if (tokens[user].exists) revert IneligibleToClaim();

            tokens[user].exists = true;
            tokens[user].isIrrevocable = isIrrevocable;

            if (isIrrevocable) {
                totalIrrevocable++;
            } else {
                totalRevocable++;
            }

            if (totalIrrevocable > irrevocableLimit || totalRevocable > revocableLimit) revert InvalidLimit();

            emit Mint(user, isIrrevocable);
        }
        function _initializeMarkets(address account) internal {
            address[] storage _allMarkets = allMarkets;
            for (uint256 i = 0; i < _allMarkets.length; ) {
                address market = _allMarkets[i];
                accrueInterest(market);

                interests[market][account].rewardIndex = markets[market].rewardIndex;
                uint256 score = _calculateScore(market, account);
                interests[market][account].score = score;
                markets[market].sumOfMembersScore = markets[market].sumOfMembersScore + score;

                unchecked {
                    i++;
                }
            }
        }

</details>

However, burning a token decrements `pendingScoreUpdates`. (Burning a token is done by withdrawing XVS from `XVSVault.sol` so that the resulting amount staked is below the minimum amount required to possess a Prime token.) Notice below:

        function _burn(address user) internal {
            ...
            _updateRoundAfterTokenBurned(user);

            emit Burn(user);
        }
        function _updateRoundAfterTokenBurned(address user) internal { 
            if (totalScoreUpdatesRequired > 0) totalScoreUpdatesRequired--;

            if (pendingScoreUpdates > 0 && !isScoreUpdated[nextScoreUpdateRoundId][user]) {
                pendingScoreUpdates--;
            }
        }

To inappropriately decrement the value of `pendingScoreUpdates`, the attacker can backrun the transaction updating the alpha/multiplier, minting and burning a Prime token (this requires the attacker to have staked the minimum amount of XVS 90 days in advance). If the number of Prime tokens minted is often at the max number of Prime tokens minted, the attacker could burn an existing token and then mint and burn a new one. Since the value of `!isScoreUpdated[nextScoreUpdateRoundId][user]` is default false, pendingScoreUpdates will be inappropriately decremented if the burned token was minted after the call to `updateMultipliers()`/`updateAlpha()`.

As aforementioned, the attacker can ensure that only other users' scores are updated until `pendingScoreUpdates` reaches zero, at which point further calls to `updateScores` will revert with the custom error `NoScoreUpdatesRequired()`.

Relevant code from `updateScores()` for reference:

        function updateScores(address[] memory users) external {
            if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired(); 
            if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

            for (uint256 i = 0; i < users.length; ) {
                ...
                pendingScoreUpdates--;
                isScoreUpdated[nextScoreUpdateRoundId][user] = true;

                unchecked {
                    i++;
                }

                emit UserScoreUpdated(user);
            }
        }

As seen, the attacker's score can avoid being updated. This is signficant if a change in multiplier or alpha would decrease the attacker's score. Because rewards are distributed according to the user's score divided by the total score, the attacker can 'freeze' their score at a higher than appropriate value and accrue increased rewards at the cost of the other users in the market.

The attacker can also prevent score updates for other users. The attacker can 'freeze' a user's score that would otherwise increase after the alpha/multiplier changes, resulting in even greater rewards accrued for the attacker and denied from other users. This is because it is possible to decrease the value of `pendingScoreUpdates` by more than one if the attacker mints and burns more than one token after the alpha/multiplier is updated.

### Math to support that a larger score results in greater reward accrual

Let $a$ represent the attacker's score if it is properly updated after a change in alpha/multiplier, $b$ represent the properly updated total score, and $c$ represent the difference between the attacker's larger unupdated score and the attacker's smaller updated score. Clearly $a$, $b$, and $c$ are positive with $a < b$. Consider the following inequality, which holds true since $a \< b$ :

$`\frac{a+c}{b+c} > \frac{a}{b} \iff a+c > \frac{a(b+c)}{b} \iff a+c > a+\frac{ac}{b}$

### Test

Paste and run the below test in the 'mint and burn' scenario in Prime.ts (line 302)

<details>

        it("prevent_Update", async () => { //test to show attacker can arbitrarily prevent multiple users from being updated by `updateScores()`
          //setup 3 users
          await prime.issue(false, [user1.getAddress(), user2.getAddress(), user3.getAddress()]);
          await xvs.connect(user1).approve(xvsVault.address, bigNumber18.mul(1000));
          await xvsVault.connect(user1).deposit(xvs.address, 0, bigNumber18.mul(1000));
          await xvs.connect(user2).approve(xvsVault.address, bigNumber18.mul(1000));
          await xvsVault.connect(user2).deposit(xvs.address, 0, bigNumber18.mul(1000));
          await xvs.connect(user3).approve(xvsVault.address, bigNumber18.mul(1000));
          await xvsVault.connect(user3).deposit(xvs.address, 0, bigNumber18.mul(1000));
          //attacker sets up addresses to mint/burn and manipulate pendingScoreUpdates
          const [,,,,user4,user5] = await ethers.getSigners();
          await xvs.transfer(user4.address, bigNumber18.mul(1000000));
          await xvs.transfer(user5.address, bigNumber18.mul(1000000));
          await xvs.connect(user4).approve(xvsVault.address, bigNumber18.mul(1000));
          await xvsVault.connect(user4).deposit(xvs.address, 0, bigNumber18.mul(1000));
          await xvs.connect(user5).approve(xvsVault.address, bigNumber18.mul(1000));
          await xvsVault.connect(user5).deposit(xvs.address, 0, bigNumber18.mul(1000));
          await mine(90 * 24 * 60 * 60);
          //change alpha, pendingScoreUpdates changed to 3
          await prime.updateAlpha(1, 5);
          //attacker backruns alpha update with minting and burning tokens, decreasing pendingScoreUpdates by 2
          await prime.connect(user4).claim();
          await xvsVault.connect(user4).requestWithdrawal(xvs.address, 0, bigNumber18.mul(1000));
          await prime.connect(user5).claim();
          await xvsVault.connect(user5).requestWithdrawal(xvs.address, 0, bigNumber18.mul(1000));
          //attacker updates user 3, decreasing pendingScoreUpdates by 1
          await prime.connect(user1).updateScores([user3.getAddress()])
          //users 1 and 2 won't be updated because pendingScoreUpdates is 0
          await expect(prime.updateScores([user1.getAddress(), user2.getAddress()])).to.be.revertedWithCustomError(prime, "NoScoreUpdatesRequired");
        });

</details>

### Tools Used

Hardhat

### Recommended Mitigation Steps

Check if `pendingScoreUpdates` is nonzero when a token is minted, and increment it if so. This removes the attacker's ability to manipulate `pendingScoreUpdates`.

```diff
    function _mint(bool isIrrevocable, address user) internal {
        if (tokens[user].exists) revert IneligibleToClaim();

        tokens[user].exists = true;
        tokens[user].isIrrevocable = isIrrevocable;

        if (isIrrevocable) { //@Gas
            totalIrrevocable++;
        } else {
            totalRevocable++;
        }

        if (totalIrrevocable > irrevocableLimit || totalRevocable > revocableLimit) revert InvalidLimit();
+       if (pendingScoreUpdates != 0) {unchecked{++pendingScoreUpdates;}} 

        emit Mint(user, isIrrevocable);
    }
```

### Further Considerations

The call to `updateMultipliers()` can be frontrun by the attacker with staking XVS and/or lending/borrowing transactions in order to increase the attacker's score before 'freezing' it.

If the attacker wants to keep the inflated score, no actions that update the attacker's score can be taken. However, the attacker can claim the outsized rewards earned at any time and as often as desired since `claimInterest()` does not update user scores.

If anyone has knowledge that the exploit has occurred, it is possible for any user's score in a market to be updated with a call to `accrueInterestAndUpdateScore()`, which can neutralize the attack.

The required amount of XVS staked for this exploit can be reduced by 1000 if this exploit is combined with the exploit titled "Irrevocable token holders can instantly mint a revocable token after burning and bypass the minimum XVS stake for revocable tokens".

**[chechu (Venus) confirmed and commented](https://github.com/code-423n4/2023-09-venus-findings/issues/555#issuecomment-1777704548):**
 > Fixed [here](https://github.com/VenusProtocol/venus-protocol/commit/dab203f882359193b3bf5ff7f1a33fec19abd468).

***

## [[H-03] Incorrect decimal usage in score calculation leads to reduced user reward earnings](https://github.com/code-423n4/2023-09-venus-findings/issues/122)
*Submitted by [Brenzee](https://github.com/code-423n4/2023-09-venus-findings/issues/122), also found by [Testerbot](https://github.com/code-423n4/2023-09-venus-findings/issues/588), [0xDetermination](https://github.com/code-423n4/2023-09-venus-findings/issues/549), [santipu\_](https://github.com/code-423n4/2023-09-venus-findings/issues/433), [ast3ros](https://github.com/code-423n4/2023-09-venus-findings/issues/410), [ether\_sky](https://github.com/code-423n4/2023-09-venus-findings/issues/387), [sces60107](https://github.com/code-423n4/2023-09-venus-findings/issues/349), [pep7siup](https://github.com/code-423n4/2023-09-venus-findings/issues/311), [Breeje](https://github.com/code-423n4/2023-09-venus-findings/issues/261), [tapir](https://github.com/code-423n4/2023-09-venus-findings/issues/141), 0xTheC0der ([1](https://github.com/code-423n4/2023-09-venus-findings/issues/91), [2](https://github.com/code-423n4/2023-09-venus-findings/issues/85)), and [SpicyMeatball](https://github.com/code-423n4/2023-09-venus-findings/issues/84)*

Users earned rewards are calculated incorrectly because of the incorrect decimals value used to calculate user's `score` and markets `sumOfMembersScore`, which impacts the `delta` that is added to market's `rewardIndex` when `Prime.accrueInterest` function is called.

### Proof of Concept

All users rewards are calculated with the following formula:

    rewards = (rewardIndex - userRewardIndex) * scoreOfUser;

This means that for user to earn rewards, market's `rewardIndex` needs to periodically increase.
 

`markets[vToken].rewardIndex` is updated (increased), when `Prime.accrueInterest` is called.

[`Prime.sol:L583-L588`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L583-L588)

```solidity
    uint256 delta;
    if (markets[vToken].sumOfMembersScore > 0) {
        delta = ((distributionIncome * EXP_SCALE) / markets[vToken].sumOfMembersScore);
    }

    markets[vToken].rewardIndex = markets[vToken].rewardIndex + delta;
```

From the code snippet above it is assumed that `markets[vToken].sumOfMembersScore` is precision of 18 decimals.

To ensure that user's `score` and markets `sumOfMemberScore` are correctly calculated, in [`Prime._calculateScore`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L647-L664) function `capital` is adjusted to 18 decimals. After that `capital` is used in `Scores.calculateScore` function.
Note: `capital` precision from [`_capitalForScore`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L872-L897) function is in precision of **underlying token decimals**.

[`Prime.sol:660-L663`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L660-L663)

```solidity
    (uint256 capital, , ) = _capitalForScore(xvsBalanceForScore, borrow, supply, market);
    capital = capital * (10 ** (18 - vToken.decimals()));

    return Scores.calculateScore(xvsBalanceForScore, capital, alphaNumerator, alphaDenominator);
```
The mistake is made when `vToken.decimals()` is used instead of `vToken.underlying().decimals()`.
 
To prove that this is the case, here are vTokens deployed on Binance Smart chain, their decimals and underlying token decimals:

| vToken                                                                        | vToken decimals | Underlying token decimals |
| ----------------------------------------------------------------------------- | --------------- | ------------------------- |
| [vUSDC](https://bscscan.com/token/0xecA88125a5ADbe82614ffC12D0DB554E2e2867C8) | 8               | 18                        |
| [vETH](https://bscscan.com/token/0xf508fcd89b8bd15579dc79a6827cb4686a3592c8)  | 8               | 18                        |
| [vBNB](https://bscscan.com/token/0xa07c5b74c9b40447a954e1466938b865b6bbea36)  | 8               | 18                        |
| [vBTC](https://bscscan.com/token/0x882c173bc7ff3b7786ca16dfed3dfffb9ee7847b)  | 8               | 18                        |
| [vUSDT](https://bscscan.com/token/0xfd5840cd36d94d7229439859c0112a4185bc0255) | 8               | 18                        |

Since `vToken.decimals()` is used, this means the precision of `capital` is `18 + (18 - 8) = 28` decimals instead of 18 decimals, which makes the `score` calculation from [`Score.calculateScore`](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/libs/Scores.sol#L22-L69) function incorrect, since the function expects `capital` to be in precision of 18 decimals.

As a result, `delta` for market's `rewardIndex` is incorrectly calculated and it can be 0 even though it shouldn't be, which means that users will not accrue any rewards.

### Update current test with correct decimals for vTokens

Developers have made a mistake when writing the tests for `Prime.sol` - in the tests they have set vToken decimals to 18 instead of 8, which makes the tests pass, but on the Binance Smart Chain all of the vToken decimals are 8.

If the decimal value of vToken is set to 8 in the tests, then the tests will fail.

Change the `vusdt`, `veth` and `vbnb` decimals to 8 and run:

```bash
npx hardhat test tests/hardhat/Prime/*.ts tests/hardhat/integration/index.ts
```

This will make the current tests fail.

### PoC Test

Here is a test where it shows, that `rewardIndex` is still 0 after `Prime.accrueInterest` is called, even though it should be > 0.

<details>
<summary>PoC test</summary>

To run the following test:

1.  Create `PoC.ts` file under the `tests/hardhat/Prime` path.
2.  Copy the code below and paste it into the `PoC.ts` file.
3.  Run `npx hardhat test tests/hardhat/Prime/PoC.ts`

```typescript
import { FakeContract, MockContract, smock } from "@defi-wonderland/smock";
import { loadFixture, mine } from "@nomicfoundation/hardhat-network-helpers";
import chai from "chai";
import { BigNumber, Signer } from "ethers";
import { ethers, upgrades } from "hardhat";

import { convertToUnit } from "../../../helpers/utils";
import {
  BEP20Harness,
  ComptrollerLens,
  ComptrollerLens__factory,
  ComptrollerMock,
  ComptrollerMock__factory,
  IAccessControlManager,
  IProtocolShareReserve,
  InterestRateModelHarness,
  PrimeLiquidityProvider,
  PrimeScenario,
  ResilientOracleInterface,
  VBep20Harness,
  XVS,
  XVSStore,
  XVSVault,
  XVSVaultScenario,
} from "../../../typechain";

const { expect } = chai;
chai.use(smock.matchers);

export const bigNumber18 = BigNumber.from("1000000000000000000"); // 1e18
export const bigNumber16 = BigNumber.from("10000000000000000"); // 1e16
export const vTokenDecimals = BigNumber.from(8);

type SetupProtocolFixture = {
  oracle: FakeContract<ResilientOracleInterface>;
  accessControl: FakeContract<IAccessControlManager>;
  comptrollerLens: MockContract<ComptrollerLens>;
  comptroller: MockContract<ComptrollerMock>;
  usdt: BEP20Harness;
  vusdt: VBep20Harness;
  eth: BEP20Harness;
  veth: VBep20Harness;
  xvsVault: XVSVaultScenario;
  xvs: XVS;
  xvsStore: XVSStore;
  prime: PrimeScenario;
  protocolShareReserve: FakeContract<IProtocolShareReserve>;
  primeLiquidityProvider: PrimeLiquidityProvider;
};

async function deployProtocol(): Promise<SetupProtocolFixture> {
  const [wallet, user1, user2, user3] = await ethers.getSigners();

  const oracle = await smock.fake<ResilientOracleInterface>("ResilientOracleInterface");
  const protocolShareReserve = await smock.fake<IProtocolShareReserve>("IProtocolShareReserve");
  const accessControl = await smock.fake<IAccessControlManager>("AccessControlManager");
  accessControl.isAllowedToCall.returns(true);
  const ComptrollerLensFactory = await smock.mock<ComptrollerLens__factory>("ComptrollerLens");
  const ComptrollerFactory = await smock.mock<ComptrollerMock__factory>("ComptrollerMock");
  const comptroller = await ComptrollerFactory.deploy();
  const comptrollerLens = await ComptrollerLensFactory.deploy();
  await comptroller._setAccessControl(accessControl.address);
  await comptroller._setComptrollerLens(comptrollerLens.address);
  await comptroller._setPriceOracle(oracle.address);
  await comptroller._setLiquidationIncentive(convertToUnit("1", 18));
  await protocolShareReserve.MAX_PERCENT.returns("100");

  const tokenFactory = await ethers.getContractFactory("BEP20Harness");
  const usdt = (await tokenFactory.deploy(
    bigNumber18.mul(100000000),
    "usdt",
    BigNumber.from(18),
    "BEP20 usdt",
  )) as BEP20Harness;

  const eth = (await tokenFactory.deploy(
    bigNumber18.mul(100000000),
    "eth",
    BigNumber.from(18),
    "BEP20 eth",
  )) as BEP20Harness;

  const wbnb = (await tokenFactory.deploy(
    bigNumber18.mul(100000000),
    "wbnb",
    BigNumber.from(18),
    "BEP20 wbnb",
  )) as BEP20Harness;

  const interestRateModelHarnessFactory = await ethers.getContractFactory("InterestRateModelHarness");
  const InterestRateModelHarness = (await interestRateModelHarnessFactory.deploy(
    BigNumber.from(18).mul(5),
  )) as InterestRateModelHarness;

  const vTokenFactory = await ethers.getContractFactory("VBep20Harness");
  const vusdt = (await vTokenFactory.deploy(
    usdt.address,
    comptroller.address,
    InterestRateModelHarness.address,
    bigNumber18,
    "VToken usdt",
    "vusdt",
    vTokenDecimals,
    wallet.address,
  )) as VBep20Harness;
  const veth = (await vTokenFactory.deploy(
    eth.address,
    comptroller.address,
    InterestRateModelHarness.address,
    bigNumber18,
    "VToken eth",
    "veth",
    vTokenDecimals,
    wallet.address,
  )) as VBep20Harness;
  const vbnb = (await vTokenFactory.deploy(
    wbnb.address,
    comptroller.address,
    InterestRateModelHarness.address,
    bigNumber18,
    "VToken bnb",
    "vbnb",
    vTokenDecimals,
    wallet.address,
  )) as VBep20Harness;

  //0.2 reserve factor
  await veth._setReserveFactor(bigNumber16.mul(20));
  await vusdt._setReserveFactor(bigNumber16.mul(20));

  oracle.getUnderlyingPrice.returns((vToken: string) => {
    if (vToken == vusdt.address) {
      return convertToUnit(1, 18);
    } else if (vToken == veth.address) {
      return convertToUnit(1200, 18);
    }
  });

  oracle.getPrice.returns((token: string) => {
    if (token == xvs.address) {
      return convertToUnit(3, 18);
    }
  });

  const half = convertToUnit("0.5", 18);
  await comptroller._supportMarket(vusdt.address);
  await comptroller._setCollateralFactor(vusdt.address, half);
  await comptroller._supportMarket(veth.address);
  await comptroller._setCollateralFactor(veth.address, half);

  await eth.transfer(user1.address, bigNumber18.mul(100));
  await usdt.transfer(user2.address, bigNumber18.mul(10000));

  await comptroller._setMarketSupplyCaps([vusdt.address, veth.address], [bigNumber18.mul(10000), bigNumber18.mul(100)]);

  await comptroller._setMarketBorrowCaps([vusdt.address, veth.address], [bigNumber18.mul(10000), bigNumber18.mul(100)]);

  const xvsFactory = await ethers.getContractFactory("XVS");
  const xvs: XVS = (await xvsFactory.deploy(wallet.address)) as XVS;

  const xvsStoreFactory = await ethers.getContractFactory("XVSStore");
  const xvsStore: XVSStore = (await xvsStoreFactory.deploy()) as XVSStore;

  const xvsVaultFactory = await ethers.getContractFactory("XVSVaultScenario");
  const xvsVault: XVSVaultScenario = (await xvsVaultFactory.deploy()) as XVSVaultScenario;

  await xvsStore.setNewOwner(xvsVault.address);
  await xvsVault.setXvsStore(xvs.address, xvsStore.address);
  await xvsVault.setAccessControl(accessControl.address);

  await xvs.transfer(xvsStore.address, bigNumber18.mul(1000));
  await xvs.transfer(user1.address, bigNumber18.mul(1000000));
  await xvs.transfer(user2.address, bigNumber18.mul(1000000));
  await xvs.transfer(user3.address, bigNumber18.mul(1000000));

  await xvsStore.setRewardToken(xvs.address, true);

  const lockPeriod = 300;
  const allocPoint = 100;
  const poolId = 0;
  const rewardPerBlock = bigNumber18.mul(1);
  await xvsVault.add(xvs.address, allocPoint, xvs.address, rewardPerBlock, lockPeriod);

  const primeLiquidityProviderFactory = await ethers.getContractFactory("PrimeLiquidityProvider");
  const primeLiquidityProvider = await upgrades.deployProxy(
    primeLiquidityProviderFactory,
    [accessControl.address, [xvs.address, usdt.address, eth.address], [10, 10, 10]],
    {},
  );

  const primeFactory = await ethers.getContractFactory("PrimeScenario");
  const prime: PrimeScenario = await upgrades.deployProxy(
    primeFactory,
    [
      xvsVault.address,
      xvs.address,
      0,
      1,
      2,
      accessControl.address,
      protocolShareReserve.address,
      primeLiquidityProvider.address,
      comptroller.address,
      oracle.address,
      10,
    ],
    {
      constructorArgs: [wbnb.address, vbnb.address, 10512000],
      unsafeAllow: ["constructor"],
    },
  );

  await xvsVault.setPrimeToken(prime.address, xvs.address, poolId);
  await prime.setLimit(1000, 1000);
  await prime.addMarket(vusdt.address, bigNumber18.mul("1"), bigNumber18.mul("1"));
  await prime.addMarket(veth.address, bigNumber18.mul("1"), bigNumber18.mul("1"));
  await comptroller._setPrimeToken(prime.address);
  await prime.togglePause();

  return {
    oracle,
    comptroller,
    comptrollerLens,
    accessControl,
    usdt,
    vusdt,
    eth,
    veth,
    xvsVault,
    xvs,
    xvsStore,
    prime,
    protocolShareReserve,
    primeLiquidityProvider,
  };
}

describe("PoC", () => {
  let deployer: Signer;
  let user1: Signer;
  let user2: Signer;
  let user3: Signer;
  let comptroller: MockContract<ComptrollerMock>;
  let prime: PrimeScenario;
  let vusdt: VBep20Harness;
  let veth: VBep20Harness;
  let usdt: BEP20Harness;
  let eth: BEP20Harness;
  let xvsVault: XVSVault;
  let xvs: XVS;
  let oracle: FakeContract<ResilientOracleInterface>;
  let protocolShareReserve: FakeContract<IProtocolShareReserve>;
  let primeLiquidityProvider: PrimeLiquidityProvider;
  let vbnb: VBep20Harness;
  let bnb: BEP20Harness;

  before(async () => {
    [deployer, user1, user2, user3] = await ethers.getSigners();
    ({
      comptroller,
      prime,
      vusdt,
      veth,
      usdt,
      eth,
      xvsVault,
      xvs,
      oracle,
      protocolShareReserve,
      primeLiquidityProvider,
    } = await loadFixture(deployProtocol));

    await protocolShareReserve.getUnreleasedFunds.returns("0");
    await protocolShareReserve.getPercentageDistribution.returns("100");

    await xvs.connect(user1).approve(xvsVault.address, bigNumber18.mul(10000));
    await xvsVault.connect(user1).deposit(xvs.address, 0, bigNumber18.mul(10000));
    await mine(90 * 24 * 60 * 60);
    await prime.connect(user1).claim();

    await xvs.connect(user2).approve(xvsVault.address, bigNumber18.mul(100));
    await xvsVault.connect(user2).deposit(xvs.address, 0, bigNumber18.mul(100));

    await eth.connect(user1).approve(veth.address, bigNumber18.mul(90));
    await veth.connect(user1).mint(bigNumber18.mul(90));

    await usdt.connect(user2).approve(vusdt.address, bigNumber18.mul(9000));
    await vusdt.connect(user2).mint(bigNumber18.mul(9000));

    await comptroller.connect(user1).enterMarkets([vusdt.address, veth.address]);

    await comptroller.connect(user2).enterMarkets([vusdt.address, veth.address]);

    await vusdt.connect(user1).borrow(bigNumber18.mul(5));
    await veth.connect(user2).borrow(bigNumber18.mul(1));

    const tokenFactory = await ethers.getContractFactory("BEP20Harness");
    bnb = (await tokenFactory.deploy(
      bigNumber18.mul(100000000),
      "bnb",
      BigNumber.from(18),
      "BEP20 bnb",
    )) as BEP20Harness;

    const interestRateModelHarnessFactory = await ethers.getContractFactory("InterestRateModelHarness");
    const InterestRateModelHarness = (await interestRateModelHarnessFactory.deploy(
      BigNumber.from(18).mul(5),
    )) as InterestRateModelHarness;

    const vTokenFactory = await ethers.getContractFactory("VBep20Harness");
    vbnb = (await vTokenFactory.deploy(
      bnb.address,
      comptroller.address,
      InterestRateModelHarness.address,
      bigNumber18,
      "VToken bnb",
      "vbnb",
      BigNumber.from(8),
      deployer.getAddress(),
    )) as VBep20Harness;

    await vbnb._setReserveFactor(bigNumber16.mul(20));
    await primeLiquidityProvider.initializeTokens([bnb.address]);

    oracle.getUnderlyingPrice.returns((vToken: string) => {
      if (vToken == vusdt.address) {
        return convertToUnit(1, 18);
      } else if (vToken == veth.address) {
        return convertToUnit(1200, 18);
      } else if (vToken == vbnb.address) {
        return convertToUnit(300, 18);
      }
    });

    oracle.getPrice.returns((token: string) => {
      if (token == xvs.address) {
        return convertToUnit(3, 18);
      }
    });

    const half = convertToUnit("0.5", 8);
    await comptroller._supportMarket(vbnb.address);
    await comptroller._setCollateralFactor(vbnb.address, half);

    bnb.transfer(user3.getAddress(), bigNumber18.mul(100));

    await comptroller._setMarketSupplyCaps([vbnb.address], [bigNumber18.mul(100)]);
    await comptroller._setMarketBorrowCaps([vbnb.address], [bigNumber18.mul(100)]);

    await bnb.connect(user3).approve(vbnb.address, bigNumber18.mul(90));
    await vbnb.connect(user3).mint(bigNumber18.mul(90));

    await vbnb.connect(user2).borrow(bigNumber18.mul(1));

    await comptroller._setPrimeToken(prime.address);
  });

  it("PoC", async () => {
    const bob = user3;
    // Bob deposits XVS in the vault
    await xvs.connect(bob).approve(xvsVault.address, bigNumber18.mul(2000));
    await xvsVault.connect(bob).deposit(xvs.address, 0, bigNumber18.mul(2000));
    await prime.issue(false, [bob.getAddress()]);
    await prime.addMarket(vbnb.address, bigNumber18.mul(1), bigNumber18.mul(1));

    // Bob mints vBNB/deposits BNB. This calls Prime.accrueInterestAndUpdateScore
    await bnb.connect(bob).approve(vbnb.address, bigNumber18.mul(90));
    await vbnb.connect(bob).mint(bigNumber18.mul(1));

    let market = await prime.markets(vbnb.address);

    // Expect that market.sumOfMembersScore is not 0. This means that the score was updated
    expect(market.sumOfMembersScore).to.not.equal(0);

    // We make the PSR.getUnreleasedFunds to return 103683. This lets Prime contract know that
    // there are unreleased funds and rewardIndex should be updated.
    await protocolShareReserve.getUnreleasedFunds.returns(103683);

    // Call accrueInterest manually.
    //
    // Since there are unreleased funds AND the sumOfMembersScore !== 0,
    // the reward index should be updated.
    await prime.accrueInterest(vbnb.address);
    market = await prime.markets(vbnb.address);

    // The reward index should be > 0, but it is not updated.
    expect(market.rewardIndex).to.equal(0);
  });
});
```
</details>

### Recommended Mitigation Steps

Make sure that underlying token decimals are used instead of vToken decimals when calculating `capital` in `Prime._calculateScore` function.

```solidity
    (uint256 capital, , ) = _capitalForScore(xvsBalanceForScore, borrow, supply, market);
    capital = capital * (10 ** (18 - IERC20Upgradeable(_getUnderlying(market)).decimals()));
```

**[chechu (Venus) confirmed via duplicate issue 588 and commented](https://github.com/code-423n4/2023-09-venus-findings/issues/588):**
 > Fixed. See [here](https://github.com/VenusProtocol/venus-protocol/commit/f1fecacada4bb5d7e4b8cad5b6ca498f2d0d16be) and [here](https://github.com/VenusProtocol/venus-protocol/commit/8ac02bc898607d775969694469e012bfbcf1a3cc)

**[0xDjango (Judge) commented](https://github.com/code-423n4/2023-09-venus-findings/issues/122#issuecomment-1793594920):**
 > Agree with high severity as the core mechanic of interest calculation is incorrect.

***
# Medium Risk Findings (3)
## [[M-01] Scores.sol: Incorrect computation of user's score when alpha is 1](https://github.com/code-423n4/2023-09-venus-findings/issues/591)
*Submitted by [Testerbot](https://github.com/code-423n4/2023-09-venus-findings/issues/591)*

The formula for a user's score depends on the `xvs` staked and the `capital`. One core variable in the calculation of a user's score is `alpha` which represents the weight for `xvs` and `capital`. It has been stated in the [documentation](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/README.md#rewards) that `alpha` can range from 0 to 1 depending on what kind of incentive the protocol wants to drive (XVS Staking or supply/borrow). Please review ***Significance of α*** subsection.

When `alpha` is 1, `xvs` has all the weight when calculating a user's score, and `capital` has no weight. If we see the Cobb-Douglas function, the value of `capital` doesn't matter, it will always return 1 since `capital^0` is always 1. So, a user does not need to borrow or lend in a given market since `capital` does not have any weight in the score calculation.

The issue is an inconsistency in the implementation of the Cobb-Douglas function.

Developers have added an exception: `if (xvs == 0 || capital == 0) return 0;`\
Because of this the code will always return 0 if either the `xvs` or the `capital` is zero, but we know this should not be the case when the `alpha` value is 1.

### PoC

To check how it works:

In `describe('boosted yield')` add the following code:

```typescript
it.only("calculate score only staking", async () => {
      const xvsBalance = bigNumber18.mul(5000);
      const capital = bigNumber18.mul(0);

      await prime.updateAlpha(1, 1); // 1

      //  5000^1 * 0^0 = 5000
      // BUT IS RETURNING 0!!!
      expect((await prime.calculateScore(xvsBalance, capital)).toString()).to.be.equal("0");
});
```
### Recommended Mitigation Steps

Only return 0 when (`xvs = 0` or `capital = 0`) &ast; and `alpha` is not 1.

**[chechu (Venus) confirmed and commented](https://github.com/code-423n4/2023-09-venus-findings/issues/591#issuecomment-1777558655):**
 > Fixed [here](https://github.com/VenusProtocol/venus-protocol/commit/64daba3f757085585a705f685e86e468580c0eb5). (Alpha cannot be 0 or 1 anymore)

**[0xDjango (Judge) commented](https://github.com/code-423n4/2023-09-venus-findings/issues/591#issuecomment-1795000916):**
 > I agree with Medium severity. In the case where Venus decides to make XVS staking the sole factor in user score, this function should calculate correctly. Also given that the documentation specifically states "The value of α is between 0-1".

***

## [[M-02] DoS and gas griefing of calls to Prime.updateScores()](https://github.com/code-423n4/2023-09-venus-findings/issues/556)
*Submitted by [0xDetermination](https://github.com/code-423n4/2023-09-venus-findings/issues/556), also found by [SBSecurity](https://github.com/code-423n4/2023-09-venus-findings/issues/682), maanas ([1](https://github.com/code-423n4/2023-09-venus-findings/issues/681), [2](https://github.com/code-423n4/2023-09-venus-findings/issues/559)), [al88nsk](https://github.com/code-423n4/2023-09-venus-findings/issues/679), [PwnStars](https://github.com/code-423n4/2023-09-venus-findings/issues/542), [Pessimistic](https://github.com/code-423n4/2023-09-venus-findings/issues/526), [dethera](https://github.com/code-423n4/2023-09-venus-findings/issues/510), [ThreeSigma](https://github.com/code-423n4/2023-09-venus-findings/issues/506), [0xblackskull](https://github.com/code-423n4/2023-09-venus-findings/issues/502), [xAriextz](https://github.com/code-423n4/2023-09-venus-findings/issues/457), [blutorque](https://github.com/code-423n4/2023-09-venus-findings/issues/438), [neumo](https://github.com/code-423n4/2023-09-venus-findings/issues/426), [tsvetanovv](https://github.com/code-423n4/2023-09-venus-findings/issues/423), [sces60107](https://github.com/code-423n4/2023-09-venus-findings/issues/334), [pina](https://github.com/code-423n4/2023-09-venus-findings/issues/274), [oakcobalt](https://github.com/code-423n4/2023-09-venus-findings/issues/232), [Breeje](https://github.com/code-423n4/2023-09-venus-findings/issues/182), [ADM](https://github.com/code-423n4/2023-09-venus-findings/issues/150), [tapir](https://github.com/code-423n4/2023-09-venus-findings/issues/88), [said](https://github.com/code-423n4/2023-09-venus-findings/issues/57), [debo](https://github.com/code-423n4/2023-09-venus-findings/issues/30), [0xweb3boy](https://github.com/code-423n4/2023-09-venus-findings/issues/24), and [Satyam\_Sharma](https://github.com/code-423n4/2023-09-venus-findings/issues/6)*

<https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200-L230> 

<https://github.com/code-423n4/2023-09-venus/blob/main/tests/hardhat/Prime/Prime.ts#L294-L301>

`updateScores()` is meant to be called to update the scores of many users after reward alpha is changed or reward multipliers are changed. An attacker can cause calls to `Prime.updateScores()` to out-of-gas revert, delaying score updates. Rewards will be distributed incorrectly until scores are properly updated.

### Proof of Concept

`updateScores()` will run out of gas and revert if any of the `users` passed in the argument array have already been updated. This is due to the `continue` statement and the incrementing location of `i`:

        function updateScores(address[] memory users) external {
            if (pendingScoreUpdates == 0) revert NoScoreUpdatesRequired();
            if (nextScoreUpdateRoundId == 0) revert NoScoreUpdatesRequired();

            for (uint256 i = 0; i < users.length; ) {
                address user = users[i];

                if (!tokens[user].exists) revert UserHasNoPrimeToken();
                if (isScoreUpdated[nextScoreUpdateRoundId][user]) continue;
                ...
                unchecked {
                    i++;
                }

                emit UserScoreUpdated(user);
            }
        }

An attacker can frontrun calls to `updateScores()` with a call to `updateScores()`, passing in a 1-member array of one of the addresses in the frontran call's argument array. The frontran call will run out of gas and revert, and only one of the users intended to be updated will actually be updated.

### Test

Paste and run the below test in the 'mint and burn' scenario in Prime.ts (line 302)

        it("dos_updateScores", async () => {
          //setup 3 users
          await prime.issue(true, [user1.getAddress(), user2.getAddress(), user3.getAddress()]);
          await xvs.connect(user1).approve(xvsVault.address, bigNumber18.mul(1000));
          await xvsVault.connect(user1).deposit(xvs.address, 0, bigNumber18.mul(1000));
          await xvs.connect(user2).approve(xvsVault.address, bigNumber18.mul(1000));
          await xvsVault.connect(user2).deposit(xvs.address, 0, bigNumber18.mul(1000));
          await xvs.connect(user3).approve(xvsVault.address, bigNumber18.mul(1000));
          await xvsVault.connect(user3).deposit(xvs.address, 0, bigNumber18.mul(1000));
          //change alpha
          await prime.updateAlpha(1, 5);
          //user 1 frontruns updateScores([many users]) with updateScores([solely user3])
          await prime.connect(user1).updateScores([user3.getAddress()])
          //frontran call should revert
          await expect(prime.updateScores([user1.getAddress(), user2.getAddress(), user3.getAddress()])).to.be.reverted;
        });

### Tools Used

Hardhat

### Recommended Mitigation Steps

Increment `i` such that `continue` doesn't result in infinite looping:

```diff
    function updateScores(address[] memory users) external {
        ...
+       for (uint256 i = 0; i < users.length; ++i} ) {
-       for (uint256 i = 0; i < users.length; ) {
            ...
-           unchecked {
-               i++;
-           }
            ...
        }
    }
```

**[0xRobocop (Lookout) commented](https://github.com/code-423n4/2023-09-venus-findings/issues/556#issuecomment-1747724530):**
 > Finding is correct. The function `updateScore` will unsuccessfully increment `i` and hence will not skip the user, causing the call to revert.
> 
> Consider QA, since the function can be called again.
> 
> Marking as primary for sponsor review anyways.

**[chechu (Venus) confirmed, but disagreed with severity and commented](https://github.com/code-423n4/2023-09-venus-findings/issues/556#issuecomment-1777561604):**
 > Consider QA.
> 
> `updateScores` could be invoked again.

**[0xDjango (Judge) commented](https://github.com/code-423n4/2023-09-venus-findings/issues/556#issuecomment-1787703933):**
 > I agree with medium severity. Take the scenario where each user update costs 10\_000 gas.
> 
> - 100 users need to be updated = `10_000 * 100` = 1\_000\_000 gas.
> - Attacker frontruns as described above, costing themselves ~10,000 gas.
> - OOG error will cause original caller to spend 1\_000\_000 gas.
> - Original caller tries again.
> - Attacker frontruns again.
> - OOG error will cause original caller to spend another 1\_000\_000 gas.
> 
> This griefing attack can cause a significant amount of required spending. 

**[chechu (Venus) commented](https://github.com/code-423n4/2023-09-venus-findings/issues/556#issuecomment-1828139845):**
 > Fixed [here](https://github.com/VenusProtocol/venus-protocol/commit/fc6a76e29c6b59a03a9ca8f5b4072aa2b9492fa7).

## [[M-03] The `owner` is a single point of failure and a centralization risk](https://gist.github.com/code423n4/9e80eddfb29953d8b5a424084a54e4ed?permalink_comment_id=4762845#m01-the-owner-is-a-single-point-of-failure-and-a-centralization-risk)
_Submitted by [Tera Bot](https://gist.github.com/code423n4/9e80eddfb29953d8b5a424084a54e4ed?permalink_comment_id=4762845#m01-the-owner-is-a-single-point-of-failure-and-a-centralization-risk)_

_Note: this finding was reported via the winning [Automated Findings report](https://gist.github.com/code423n4/9e80eddfb29953d8b5a424084a54e4ed). It was declared out of scope for the audit, but is being included here for completeness._

Having a single EOA as the only owner of contracts is a large centralization risk and a single point of failure. A single private key may be taken in a hack, or the sole holder of the key may become unable to retrieve the key when necessary. Consider changing to a multi-signature setup, or having a role-based authorization model.

*There are 3 instances of this issue:*
```solidity
File: contracts/Tokens/Prime/PrimeLiquidityProvider.sol

118         function initializeTokens(address[] calldata tokens_) external onlyOwner {

177         function setPrimeToken(address prime_) external onlyOwner {

216         function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {

```
*GitHub*: [[118](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L118-L118), [177](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L177-L177), [216](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216-L216)]

**[chechu (Venus) acknowledged and commented](https://gist.github.com/code423n4/9e80eddfb29953d8b5a424084a54e4ed?permalink_comment_id=4762845#gistcomment-4762845):**
 >Regarding [M‑03] The owner is a single point of failure and a centralization risk, the owner won't be an EOA, but that cannot be specified in the solidity code. The owner of our contracts is always the Normal Timelock contract deployed at 0x939bD8d64c0A9583A7Dcea9933f7b21697ab6396. This contract is used in the Governance process, so after a voting period of 24 hours, and an extra delay of 48 hours if the vote passed, this contract will execute the commands agreed on the Venus Improvement Proposal.

**[0xDjango (Judge) commented](https://gist.github.com/code423n4/9e80eddfb29953d8b5a424084a54e4ed?permalink_comment_id=4762962#gistcomment-4762962):**
 >Confirming medium severity for centralization risk, though Venus does has safeguards in place to ensure that decentralization is achieved through governance. From a user-perspective, the potentiality of centralization risks must be highlighted.

***

# Low Risk and Non-Critical Issues

For this audit, 88 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-09-venus-findings/issues/639) by **Bauchibred** received the top score from the judge.

*The following wardens also submitted reports: [0xprinc](https://github.com/code-423n4/2023-09-venus-findings/issues/634), [0xDetermination](https://github.com/code-423n4/2023-09-venus-findings/issues/513), [DavidGiladi](https://github.com/code-423n4/2023-09-venus-findings/issues/437), [josephdara](https://github.com/code-423n4/2023-09-venus-findings/issues/291), [0xTheC0der](https://github.com/code-423n4/2023-09-venus-findings/issues/97), [Testerbot](https://github.com/code-423n4/2023-09-venus-findings/issues/664), [0xScourgedev](https://github.com/code-423n4/2023-09-venus-findings/issues/654), [Maroutis](https://github.com/code-423n4/2023-09-venus-findings/issues/641), [inzinko](https://github.com/code-423n4/2023-09-venus-findings/issues/611), [ge6a](https://github.com/code-423n4/2023-09-venus-findings/issues/608), [merlin](https://github.com/code-423n4/2023-09-venus-findings/issues/602), [rokinot](https://github.com/code-423n4/2023-09-venus-findings/issues/593), [0xMosh](https://github.com/code-423n4/2023-09-venus-findings/issues/566), [btk](https://github.com/code-423n4/2023-09-venus-findings/issues/561), [bin2chen](https://github.com/code-423n4/2023-09-venus-findings/issues/548), [0xdice91](https://github.com/code-423n4/2023-09-venus-findings/issues/546), [Fulum](https://github.com/code-423n4/2023-09-venus-findings/issues/531), [PwnStars](https://github.com/code-423n4/2023-09-venus-findings/issues/521), [gkrastenov](https://github.com/code-423n4/2023-09-venus-findings/issues/518), [ThreeSigma](https://github.com/code-423n4/2023-09-venus-findings/issues/511), [imare](https://github.com/code-423n4/2023-09-venus-findings/issues/504), [Aymen0909](https://github.com/code-423n4/2023-09-venus-findings/issues/500), [Flora](https://github.com/code-423n4/2023-09-venus-findings/issues/485), [DeFiHackLabs](https://github.com/code-423n4/2023-09-venus-findings/issues/482), [SPYBOY](https://github.com/code-423n4/2023-09-venus-findings/issues/478), [KrisApostolov](https://github.com/code-423n4/2023-09-venus-findings/issues/477), [J4X](https://github.com/code-423n4/2023-09-venus-findings/issues/458), [neumo](https://github.com/code-423n4/2023-09-venus-findings/issues/456), [jnforja](https://github.com/code-423n4/2023-09-venus-findings/issues/446), [alexweb3](https://github.com/code-423n4/2023-09-venus-findings/issues/445), [Mirror](https://github.com/code-423n4/2023-09-venus-findings/issues/440), [berlin-101](https://github.com/code-423n4/2023-09-venus-findings/issues/429), [deth](https://github.com/code-423n4/2023-09-venus-findings/issues/425), [pina](https://github.com/code-423n4/2023-09-venus-findings/issues/424), [0xTiwa](https://github.com/code-423n4/2023-09-venus-findings/issues/422), [d3e4](https://github.com/code-423n4/2023-09-venus-findings/issues/421), [twicek](https://github.com/code-423n4/2023-09-venus-findings/issues/412), [ast3ros](https://github.com/code-423n4/2023-09-venus-findings/issues/411), [tonisives](https://github.com/code-423n4/2023-09-venus-findings/issues/407), [0xpiken](https://github.com/code-423n4/2023-09-venus-findings/issues/382), [MohammedRizwan](https://github.com/code-423n4/2023-09-venus-findings/issues/344), [ether\_sky](https://github.com/code-423n4/2023-09-venus-findings/issues/338), [pep7siup](https://github.com/code-423n4/2023-09-venus-findings/issues/336), [Brenzee](https://github.com/code-423n4/2023-09-venus-findings/issues/309), [Tricko](https://github.com/code-423n4/2023-09-venus-findings/issues/306), [jkoppel](https://github.com/code-423n4/2023-09-venus-findings/issues/297), [lsaudit](https://github.com/code-423n4/2023-09-venus-findings/issues/290), [0xfusion](https://github.com/code-423n4/2023-09-venus-findings/issues/271), [glcanvas](https://github.com/code-423n4/2023-09-venus-findings/issues/264), [lotux](https://github.com/code-423n4/2023-09-venus-findings/issues/258), [santipu\_](https://github.com/code-423n4/2023-09-venus-findings/issues/257), [xAriextz](https://github.com/code-423n4/2023-09-venus-findings/issues/254), [y4y](https://github.com/code-423n4/2023-09-venus-findings/issues/248), [blutorque](https://github.com/code-423n4/2023-09-venus-findings/issues/242), [ArmedGoose](https://github.com/code-423n4/2023-09-venus-findings/issues/240), [sashik\_eth](https://github.com/code-423n4/2023-09-venus-findings/issues/234), [Krace](https://github.com/code-423n4/2023-09-venus-findings/issues/233), [Daniel526](https://github.com/code-423n4/2023-09-venus-findings/issues/204), [al88nsk](https://github.com/code-423n4/2023-09-venus-findings/issues/203), [Norah](https://github.com/code-423n4/2023-09-venus-findings/issues/201), [HChang26](https://github.com/code-423n4/2023-09-venus-findings/issues/188), [squeaky\_cactus](https://github.com/code-423n4/2023-09-venus-findings/issues/186), [joaovwfreire](https://github.com/code-423n4/2023-09-venus-findings/issues/185), [Breeje](https://github.com/code-423n4/2023-09-venus-findings/issues/177), [oakcobalt](https://github.com/code-423n4/2023-09-venus-findings/issues/167), [hals](https://github.com/code-423n4/2023-09-venus-findings/issues/161), [rvierdiiev](https://github.com/code-423n4/2023-09-venus-findings/issues/140), [peanuts](https://github.com/code-423n4/2023-09-venus-findings/issues/131), [mahdirostami](https://github.com/code-423n4/2023-09-venus-findings/issues/129), [vagrant](https://github.com/code-423n4/2023-09-venus-findings/issues/128), [nobody2018](https://github.com/code-423n4/2023-09-venus-findings/issues/120), [nisedo](https://github.com/code-423n4/2023-09-venus-findings/issues/117), [IceBear](https://github.com/code-423n4/2023-09-venus-findings/issues/100), [terrancrypt](https://github.com/code-423n4/2023-09-venus-findings/issues/83), [Hama](https://github.com/code-423n4/2023-09-venus-findings/issues/81), [said](https://github.com/code-423n4/2023-09-venus-findings/issues/79), [n1punp](https://github.com/code-423n4/2023-09-venus-findings/issues/76), [nadin](https://github.com/code-423n4/2023-09-venus-findings/issues/70), [e0d1n](https://github.com/code-423n4/2023-09-venus-findings/issues/58), [kutugu](https://github.com/code-423n4/2023-09-venus-findings/issues/53), [seerether](https://github.com/code-423n4/2023-09-venus-findings/issues/51), [0xWaitress](https://github.com/code-423n4/2023-09-venus-findings/issues/42), [0x3b](https://github.com/code-423n4/2023-09-venus-findings/issues/28), [0xweb3boy](https://github.com/code-423n4/2023-09-venus-findings/issues/25), [TangYuanShen](https://github.com/code-423n4/2023-09-venus-findings/issues/22), [orion](https://github.com/code-423n4/2023-09-venus-findings/issues/10), and [ptsanev](https://github.com/code-423n4/2023-09-venus-findings/issues/5).*

## [01] Prime.sol's `_claimInterest()` should apply a slippage or allow users to partially claim their interest

Medium to low impact. The `_claimInterest` function hardcodes a 0% percent slippage and does not perform a final check after attempting to release funds from various sources. If the contract's balance is still insufficient after these fund releases, it could lead to a revert on the `safeTransfer` call, preventing users from claiming their interest.

### Proof of Concept

Take a look at the `_claimInterest()` function:

```solidity
function _claimInterest(address vToken, address user) internal returns (uint256) {
  uint256 amount = getInterestAccrued(vToken, user);
  amount += interests[vToken][user].accrued;

  interests[vToken][user].rewardIndex = markets[vToken].rewardIndex;
  interests[vToken][user].accrued = 0;

  address underlying = _getUnderlying(vToken);
  IERC20Upgradeable asset = IERC20Upgradeable(underlying);

  if (amount > asset.balanceOf(address(this))) {
    address[] memory assets = new address[](1);
    assets[0] = address(asset);
    IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
    if (amount > asset.balanceOf(address(this))) {
      IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
      //@audit-issue M no extra check to see if the balance is still not enough, in the case where it isn't then the attempt to safetransfer the tokens  would always revert so at least available balance could be sent to user and then later on extras get sent, no?
      unreleasedPLPIncome[underlying] = 0;
    }
  }

  asset.safeTransfer(user, amount);

  emit InterestClaimed(user, vToken, amount);

  return amount;
}
```

In the above function, the contract tries to ensure it has enough balance to pay out the user's interest in the following way:

1. If the required `amount` is greater than the contract's balance, it releases funds from `IProtocolShareReserve`.
2. If the amount is still greater, it attempts to release funds from `IPrimeLiquidityProvider`.

The potential issue arises after this. There is no final check to ensure that the `amount` is less than or equal to the contract's balance before executing `safeTransfer`, do note that a slippage of 0% has been applied.

If both releases (from `IProtocolShareReserve` and `IPrimeLiquidityProvider`) do not provide sufficient tokens, the `safeTransfer` will revert, and the user will be unable to claim their interest.

Depending on the situation this might not only lead to user frustration but also cause trust issues with the platform and would lead to user funds to be stuck s

### Recommended Mitigation Steps

To address this issue, the following changes are suggested:

- Add a final balance check.
- If the balance is still insufficient, send whatever is available to the user.
- Store the deficit somewhere and inform the user that there's more to claim once the contract has sufficient funds.

A diff for the recommended changes might look like:

```diff
function _claimInterest(address vToken, address user) internal returns (uint256) {
    ...
    if (amount > asset.balanceOf(address(this))) {
        address[] memory assets = new address[](1);
        assets[0] = address(asset);
        IProtocolShareReserve(protocolShareReserve).releaseFunds(comptroller, assets);
        if (amount > asset.balanceOf(address(this))) {
            IPrimeLiquidityProvider(primeLiquidityProvider).releaseFunds(address(asset));
            unreleasedPLPIncome[underlying] = 0;
        }
    }

+   uint256 availableBalance = asset.balanceOf(address(this));
+   uint256 transferAmount = (amount <= availableBalance) ? amount : availableBalance;

-   asset.safeTransfer(user, amount);
+   asset.safeTransfer(user, transferAmount);

    emit InterestClaimed(user, vToken, transferAmount);

    return transferAmount;
}
```

This ensures that a user always gets whatever is available, if the third suggestion is going to be implemented then do not forget to store the differences in a variable

> NB: Submitting as QA, due to the chances of this happening being relatively low, but leaving the final decision to judge to upgrade to Medium if they see fit.

## [02] `getEffectiveDistributionSpeed()` should prevent reversion and return `0` if `accrued` is ever more than `balance`

Low impact, since this is just how to better implement `getEffectiveDistributionSpeed()` to cover more valid _edge cases_.

### Proof of Concept

Take a look at `getEffectiveDistributionSpeed()`:

```solidity
function getEffectiveDistributionSpeed(address token_) external view returns (uint256) {
  uint256 distributionSpeed = tokenDistributionSpeeds[token_];
  uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
  uint256 accrued = tokenAmountAccrued[token_];

  if (balance - accrued > 0) {
    return distributionSpeed;
  }

  return 0;
}
```

As seen, the above function is used to get the rewards per block for a specific token; the issue is that an assumption is made that `balance >= accrued` is always true, which is not the case.

To support my argument above, note that the `PrimeLiquidityProvider.sol` also employs functions like `sweepTokens()` and `releaseFunds`. Whereas the latter makes an update to the accrual during its execution (i.e., setting the `tokenAmountAccrued[token_]` to `0`), the former does not. This can be seen below:

```solidity
function sweepToken(IERC20Upgradeable token_, address to_, uint256 amount_) external onlyOwner {
  uint256 balance = token_.balanceOf(address(this));
  if (amount_ > balance) {
    revert InsufficientBalance(amount_, balance);
  }

  emit SweepToken(address(token_), to_, amount_);

  token_.safeTransfer(to_, amount_);
}

function releaseFunds(address token_) external {
  if (msg.sender != prime) revert InvalidCaller();
  if (paused()) {
    revert FundsTransferIsPaused();
  }

  accrueTokens(token_);
  uint256 accruedAmount = tokenAmountAccrued[token_];
  tokenAmountAccrued[token_] = 0;

  emit TokenTransferredToPrime(token_, accruedAmount);

  IERC20Upgradeable(token_).safeTransfer(prime, accruedAmount);
}
```

Now, in a case where a portion of a token has been swept and the accrual is now more than the balance of said token, the `getEffectiveDistributionSpeed()` function reverts due to an underflow from the check here:

```solidity
  uint256 distributionSpeed = tokenDistributionSpeeds[token_];
  uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
  uint256 accrued = tokenAmountAccrued[token_];

  if (balance - accrued > 0)
```

### Recommended Mitigation Steps

Make an addition to cover up the edge case in the `getEffectiveDistributionSpeed()` function, i.e., if accrued is already more than the balance then the distribution speed should be the same as if the difference is 0.

An approach to the recommended fix can be seen from the diff below:

```diff
    function getEffectiveDistributionSpeed(address token_) external view returns (uint256) {
        uint256 distributionSpeed = tokenDistributionSpeeds[token_];
        uint256 balance = IERC20Upgradeable(token_).balanceOf(address(this));
        uint256 accrued = tokenAmountAccrued[token_];
-        if (balance - accrued > 0) {
+        if (balance > accrued) {
            return distributionSpeed;
        }

        return 0;
    }
```
## [03] Inconsistency in parameter validations between sister functions

Impact is low/informational.

The inconsistency in validation checks between similar functions can lead to unexpected behavior.

### Proof of Concept

Take a look at `updateAlpha()`:

<details>

```solidity
function updateAlpha(uint128 _alphaNumerator, uint128 _alphaDenominator) external {
  _checkAccessAllowed("updateAlpha(uint128,uint128)");
  _checkAlphaArguments(_alphaNumerator, _alphaDenominator);

  emit AlphaUpdated(alphaNumerator, alphaDenominator, _alphaNumerator, _alphaDenominator);

  alphaNumerator = _alphaNumerator;
  alphaDenominator = _alphaDenominator;

  for (uint256 i = 0; i < allMarkets.length; ) {
    accrueInterest(allMarkets[i]);

    unchecked {
      i++;
    }
  }

  _startScoreUpdateRound();
}

function updateMultipliers(address market, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
  _checkAccessAllowed("updateMultipliers(address,uint256,uint256)");
  if (!markets[market].exists) revert MarketNotSupported();

  accrueInterest(market);

  emit MultiplierUpdated(
    market,
    markets[market].supplyMultiplier,
    markets[market].borrowMultiplier,
    supplyMultiplier,
    borrowMultiplier
  );
  markets[market].supplyMultiplier = supplyMultiplier;
  markets[market].borrowMultiplier = borrowMultiplier;

  _startScoreUpdateRound();
}
```
</details>

In the `updateAlpha` function, there is a check `_checkAlphaArguments(_alphaNumerator, _alphaDenominator)` which ensures the correctness of the arguments provided. This establishes a certain standard of validation for updating alpha values.

```solidity
function _checkAlphaArguments(uint128 _alphaNumerator, uint128 _alphaDenominator) internal {
  if (_alphaDenominator == 0 || _alphaNumerator > _alphaDenominator) {
    revert InvalidAlphaArguments();
  }
}
```

However, in the `updateMultipliers` function, there is no equivalent validation function like `_checkAlphaArguments`. This discrepancy implies that the two functions, which are logically do not maintain a consistent standard of input validation.

### Recommended Mitigation Steps

Consider introducing a validation function (e.g., `_checkMultipliersArguments`) that will validate the parameters `supplyMultiplier` and `borrowMultiplier`.

- Incorporate this validation function into the `updateMultipliers` function, similar to how `_checkAlphaArguments` is used in the `updateAlpha` function.
- Make sure that the validation function checks for logical constraints, edge cases, or any other relevant criteria for these parameters.

A diff for the recommended changes might appear as:

```diff
+ function _checkMultipliersArguments(uint256 supplyMultiplier, uint256 borrowMultiplier) internal {
+     // Implement the validation logic here
+     ...
+ }

function updateMultipliers(address market, uint256 supplyMultiplier, uint256 borrowMultiplier) external {
    ...
+   _checkMultipliersArguments(supplyMultiplier, borrowMultiplier);
    ...
    if (!markets[market].exists) revert MarketNotSupported();
    ...
    emit MultiplierUpdated(
        ...
    );
    markets[market].supplyMultiplier = supplyMultiplier;
    markets[market].borrowMultiplier = borrowMultiplier;

    _startScoreUpdateRound();
}
```

## [04] The `lastAccruedBlock` naming needs to be refactored

Low impact, confusing comment/docs since the `lastAccruedBlock` is either wrongly implemented or wrongly named.

### Proof of Concept

Take a look at [PrimeLiquidityProvider.sol#L22-L24](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L22-L24):

```soldidity
    /// @notice The rate at which token is distributed to the Prime contract
    mapping(address => uint256) public lastAccruedBlock;
```

As seen, `lastAccruedBlock` currently claims to be the rate at which a token is distributed to the prime contract, but this is wrong since this mapping actuallly holds value for the last block an accrual was made for an address.

### Recommended Mitigation Steps

Fix the comments around the `lastAccruedBlock` mapping or change the name itself.

## [05] More sanity checks should be applied to `calculateScore()`

Low impact, since the chances of passing a valuse above type(int256).max is relatively slim.

### Proof of Concept

Take a look at `calculateScore()`:

<details>

```solidity
function calculateScore(
  uint256 xvs,
  uint256 capital,
  uint256 alphaNumerator,
  uint256 alphaDenominator
) internal pure returns (uint256) {
  // Score function is:
  // xvs^𝝰 * capital^(1-𝝰)
  //    = capital * capital^(-𝝰) * xvs^𝝰
  //    = capital * (xvs / capital)^𝝰
  //    = capital * (e ^ (ln(xvs / capital))) ^ 𝝰
  //    = capital * e ^ (𝝰 * ln(xvs / capital))     (1)
  // or
  //    = capital / ( 1 / e ^ (𝝰 * ln(xvs / capital)))
  //    = capital / (e ^ (𝝰 * ln(xvs / capital)) ^ -1)
  //    = capital / e ^ (𝝰 * -1 * ln(xvs / capital))
  //    = capital / e ^ (𝝰 * ln(capital / xvs))     (2)
  //
  // To avoid overflows, use (1) when xvs < capital and
  // use (2) when capital < xvs

  // If any side is 0, exit early
  if (xvs == 0 || capital == 0) return 0;

  // If both sides are equal, we have:
  // xvs^𝝰 * capital^(1-𝝰)
  //    = xvs^𝝰 * xvs^(1-𝝰)
  //    = xvs^(𝝰 + 1 - 𝝰)     = xvs
  if (xvs == capital) return xvs;

  bool lessxvsThanCapital = xvs < capital;

  // (xvs / capital) or (capital / xvs), always in range (0, 1)
  int256 ratio = lessxvsThanCapital ? FixedMath.toFixed(xvs, capital) : FixedMath.toFixed(capital, xvs);

  // e ^ ( ln(ratio) * 𝝰 )
  int256 exponentiation = FixedMath.exp(
    (FixedMath.ln(ratio) * alphaNumerator.toInt256()) / alphaDenominator.toInt256()
  );

  if (lessxvsThanCapital) {
    return FixedMath.uintMul(capital, exponentiation);
  }

  // capital / e ^ (𝝰 * ln(capital / xvs))
  return FixedMath.uintDiv(capital, exponentiation);
}
```

</details>

Do note that the function is used to calculate a membership score given some amount of `xvs` and `capital`, issue is that both alpha numerators and denoimantors are passed in as `uint256` values and then while getting the `exponentiation` value these values are converted to `int256` which would cause a revert.

### Recommended Mitigation Steps

Better code structure could be applied, from the get go the values passed in for numerators and denominators could be checked to ensure that they are not above the max value of int256.

## [06] The `getPendingInterests()` should be better implemented to handle cases where a market is unavailable

Low impact. If one market goes down or becomes unresponsive, users won't be able to retrieve their pending interests for any of the subsequent markets. This can lead to misinformation and might affect user decisions.

### Proof of Concept

Take a look at`getPendingInterests()`:

```solidity
function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) {
  address[] storage _allMarkets = allMarkets;
  PendingInterest[] memory pendingInterests = new PendingInterest[](_allMarkets.length);
  //@audit-ok this function currently does not take into account that one of the markets could go down in that case a break in this loop could occur, check perennial sherlock

  for (uint256 i = 0; i < _allMarkets.length; ) {
    address market = _allMarkets[i];
    uint256 interestAccrued = getInterestAccrued(market, user);
    uint256 accrued = interests[market][user].accrued;

    pendingInterests[i] = PendingInterest({ market: IVToken(market).underlying(), amount: interestAccrued + accrued });

    unchecked {
      i++;
    }
  }

  return pendingInterests;
}
```

As seen, there's a loop that goes through all available markets to fetch the boosted pending interests for a user. However, if any of these calls (like `getInterestAccrued`) fail due to market unavailability or any other issue, the loop will be interrupted.

### Recommended Mitigation Steps

To ensure the function's resilience and provide accurate information even if some markets are down, wrap the calls inside the loop in a `try/catch` block. If a call fails, log an event indicating the market that caused the failure:

```solidity
event MarketCallFailed(address indexed market);

for (uint256 i = 0; i < _allMarkets.length; ) {
    address market = _allMarkets[i];

    try {
        uint256 interestAccrued = getInterestAccrued(market, user);
        uint256 accrued = interests[market][user].accrued;

        pendingInterests[i] = PendingInterest({
            market: IVToken(market).underlying(),
            amount: interestAccrued + accrued
        });
    } catch {
        emit MarketCallFailed(market);
    }

    unchecked {
        i++;
    }
}
```

By incorporating this change, even if a market fails, the loop won't break, and the function will continue to retrieve the pending interests for the remaining markets. The emitted event will notify off-chain systems or users about the problematic market.

## [07] Missing zero/isContract checks in important functions

Low impact, since this requires a bit of an admin error, but some functions which could be more secure using the `_ensureZeroAddress()` currently do not implement this and could lead to issues.

### Proof of Concept

Using the `PrimeLiquidityProvider.sol` in scope as acase study.
Below is the implementation of `_ensureZeroAddress()`

```solidity
function _ensureZeroAddress(address address_) internal pure {
  if (address_ == address(0)) {
    revert InvalidArguments();
  }
}
```

As seen the above is used within protocol as a modifier/function to revert whenever addresses are being passed, this can be seen to be implemented in the `setPrime()` function and others, but that's not always the case and in some instances addresses are not ensured to not be zero.

Additionally, as a security measure an `isContract()` function could be added and used to check for instances where the provided addresses must be valid contracts with code.

### Recommended Mitigation Steps

Make use of `_ensureZeroAddress()` in all instances where addresses could be passed.

If the `isContract()` is going to be implemented then the functionality to be added could look something like this:

```solidity
function isContract(address addr) internal view returns (bool) {
  uint256 size;
  assembly {
    size := extcodesize(addr)
  }
  return size > 0;
}
```

And then, this could be applied to instances where addreses must have byte code.

## [08] The `uintDiv()` function from `FixedMath.sol` should protect against division by `0`

Low impact, just info on how to prevent panic errors cause by `division by zero`

### Proof of Concept

Take a look at `uintDiv()`:

```solidity
function uintDiv(uint256 u, int256 f) internal pure returns (uint256) {
  if (f < 0) revert InvalidFixedPoint(); //@audit
  // multiply `u` by FIXED_1 to cancel out the built-in FIXED_1 in f
  return uint256((u.toInt256() * FixedMath0x.FIXED_1) / f);
}
```

As seen, this function is used to divide an unsigned `int` in this case `u` by a fixed number `f`, but currently the only check applied to the `uintDiv()` function in regards to the divisor `f` is:
`        if (f < 0) revert InvalidFixedPoint();`
The above means that a value of `0` would get accepted for `f` which would result in a panic error.

### Recommended Mitigation Steps

Make changes to the `uintDiv()` function

```diff
    function uintDiv(uint256 u, int256 f) internal pure returns (uint256) {
-        if (f < 0) revert InvalidFixedPoint();
+        if (f <= 0) revert InvalidFixedPoint();
        // multiply `u` by FIXED_1 to cancel out the built-in FIXED_1 in f
        return uint256((u.toInt256() * FixedMath0x.FIXED_1) / f);
    }
```

## [09] Fix typo in docs

Low impact, info on how to have a better code documentation.

### Proof of Concept

Take a look at the Prime token's [ReadMe](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/README.md?plain=1#L295-L296)

Line 296 states:

```
The OpenZeppelin Plausable contract is used. Only the `claimInterest` function is under control of this pause mechanism.
```

As seen an evident typo has been missed.

### Recommended Mitigation Steps

Change the line, from:

```
The OpenZeppelin Plausable contract is used. Only the `claimInterest` function is under control of this pause mechanism.
```

To:

```
The OpenZeppelin Pausable contract is used. Only the `claimInterest` function is under control of this pause mechanism.
```

**[0xDjango (Judge) commented](https://github.com/code-423n4/2023-09-venus-findings/issues/639#issuecomment-1795027936):**
 > Agree with all QA points, though a couple require design changes that may outweigh the benefit to the protocol from an added risk standpoint.

 **[chechu (Venus) commented](https://github.com/code-423n4/2023-09-venus-findings/issues/639#issuecomment-1828144005):**
 > 1. The warden doesn't explain how Prime can get from the sources less funds than needed. Accounting avoids that scenario. Fee-on-transfer tokens could generate this issue, but they shouldn't be added as Prime market.
 When claimInterest() is called we transfer tokens from PrimeLiquidityProvider to Prime contract when there is insufficient funds. So users can always claim full rewards. 

 >2. Fixed [here](https://github.com/VenusProtocol/venus-protocol/commit/b0ff7ba0f5e304f6b88bedfc13b20d125dc026e6).

 >3. Acknowledged.

 >4. Fixed [here](https://github.com/VenusProtocol/venus-protocol/commit/a567d94a7f79f257ab759ff6f154f6f258c2ce13).

 >5. Acknowledged.

 >6. Acknowledged.

 >7. Partially fixed [here](https://github.com/VenusProtocol/venus-protocol/commit/df9e9af4b009bf138dfce8965f07f1c555b3686a).

 >8. Acknowledged.

 >9. Fixed [here](https://github.com/VenusProtocol/venus-protocol-documentation/pull/120/commits/5aad7c85da0824f9e0bbc7d699e318ff424340e1).



***

# Gas Optimizations

For this audit, 11 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-09-venus-findings/issues/357) by **DavidGiladi** received the top score from the judge.

*The following wardens also submitted reports: [0xhacksmithh](https://github.com/code-423n4/2023-09-venus-findings/issues/658), [0xprinc](https://github.com/code-423n4/2023-09-venus-findings/issues/635), [lsaudit](https://github.com/code-423n4/2023-09-venus-findings/issues/285), [pavankv](https://github.com/code-423n4/2023-09-venus-findings/issues/64), [pontifex](https://github.com/code-423n4/2023-09-venus-findings/issues/655), [hihen](https://github.com/code-423n4/2023-09-venus-findings/issues/571), [jkoppel](https://github.com/code-423n4/2023-09-venus-findings/issues/192), [oakcobalt](https://github.com/code-423n4/2023-09-venus-findings/issues/171), [0x3b](https://github.com/code-423n4/2023-09-venus-findings/issues/27), and [0xWaitress](https://github.com/code-423n4/2023-09-venus-findings/issues/3).*

## [G-01] Avoid unnecessary storage updates
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 2400

### Note 
I reported only on three issues that were missing in the winning bot.

### Description
Avoid updating storage when the value hasn't changed. If the old value is equal to the new value, not re-storing the value will avoid a `SSTORE` operation (costing 2900 gas), potentially at the expense of a `SLOAD` operation (2100 gas) or a `WARMACCESS` operation (100 gas).

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#
The function `updateAlpha()` changes the state variable without first verifying if the values are different.


```
File: contracts/Tokens/Prime/Prime.sol

243    alphaNumerator = _alphaNumerator
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L243](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L243)


#
The function `updateAlpha()` changes the state variable without first verifying if the values are different.


```
File: contracts/Tokens/Prime/Prime.sol

244    alphaDenominator = _alphaDenominator
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L244](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L244)


#
The function `updateAssetsState()` changes the state variable without first verifying if the values are different.


```
File: contracts/Tokens/Prime/Prime.sol

460    unreleasedPSRIncome[_getUnderlying(address(market))] = 0
```

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L460](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L460)


</details>

# 


## [G-02] Multiplication and Division by 2 Should use in Bit Shifting
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 100

### Description
The expressions 'x * 2' and 'x / 2' can be optimized for gas efficiency by utilizing bitwise operations. In Solidity, you can achieve the same results by using bitwise left shift (x << 1) for multiplication and bitwise right shift (x >> 1) for division.

Using bitwise shift operations (SHL and SHR) instead of multiplication (MUL) and division (DIV) opcodes can lead to significant gas savings. The MUL and DIV opcodes cost 5 gas, while the SHL and SHR opcodes incur a lower cost of only 3 gas.

By leveraging these more efficient bitwise operations, you can reduce the gas consumption of your smart contracts and enhance their overall performance.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

122    r += (z * (0x100000000000000000000000000000000 - y)) / 0x100000000000000000000000000000000
```
 instead `340282366920938463463374607431768211456` use bit shifting `128` 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L122](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L122)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

124    r += (z * (0x0aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa - y)) / 0x200000000000000000000000000000000
```
 instead `680564733841876926926749214863536422912` use bit shifting `129` 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L124](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L124)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

128    r += (z * (0x092492492492492492492492492492492 - y)) / 0x400000000000000000000000000000000
```
 instead `1361129467683753853853498429727072845824` use bit shifting `130` 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L128](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L128)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

136    r += (z * (0x088888888888888888888888888888888 - y)) / 0x800000000000000000000000000000000
```
 instead `2722258935367507707706996859454145691648` use bit shifting `131` 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L136](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L136)


#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

200    r += z * 0x0000000000000001
```
 instead `1` use bit shifting `0` 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L200](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L200)


</details>

# 


## [G-03] Modulus operations that could be unchecked
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 85

### Description
Modulus operations should be unchecked to save gas since they cannot overflow or underflow. Execution of modulus operations outside `unchecked` blocks adds nothing but overhead. Saves about 30 gas.

<details>

<summary>
There is 1 instance of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/libs/FixedMath0x.sol

162    x % 0x0000000000000000000000000000000010000000000000000000000000000000
```
 should be unchecked

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L162](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath0x.sol#L162)


</details>

# 


## [G-04] Short-circuit rules can be used to optimize some gas usage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 6300

### Description
Some conditions may be reordered to save an SLOAD (2100 gas), as we avoid reading state variables when the first part of the condition fails (with &&), or succeeds (with ||). For instance, consider a scenario where you have a `stateVariable` (a variable stored in contract storage) and a `localVariable` (a variable in memory). 

If you have a condition like `stateVariable > 0 && localVariable > 0`, if `localVariable > 0` is false, the Solidity runtime will still execute `stateVariable > 0`, which costs an SLOAD operation (2100 gas). However, if you reorder the condition to `localVariable > 0 && stateVariable > 0`, the `stateVariable > 0` check won't happen if `localVariable > 0` is false, saving you the SLOAD gas cost.

Similarly, for the `||` operator, if you have a condition like `stateVariable > 0 || localVariable > 0`, and `stateVariable > 0` is true, the Solidity runtime will still execute `localVariable > 0`. But if you reorder the condition to `localVariable > 0 || stateVariable > 0`, and `localVariable > 0` is true, the `stateVariable > 0` check won't happen, again saving you the SLOAD gas cost.

This detector checks for such conditions in the contract and reports if any condition could be optimized by taking advantage of the short-circuiting behavior of `&&` and `||`.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

377    stakedAt[user] == 0 && isAccountEligible && !tokens[user].exists
```


```
 // @audit: Switch isAccountEligible && stakedAt[user] == 0 
```
[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L377](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L377)


#

```
File: contracts/Tokens/Prime/Prime.sol

379    tokens[user].exists && isAccountEligible
```


```
 // @audit: Switch isAccountEligible && tokens[user].exists 
```
[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L379](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L379)


#

```
File: contracts/Tokens/Prime/Prime.sol

369    tokens[user].exists && !isAccountEligible
```


```
 // @audit: Switch ! isAccountEligible && tokens[user].exists 
```
[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L369](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L369)


</details>

# 


## [G-05] Unnecessary Casting of Variables
- Severity: Gas Optimization
- Confidence: High

### Description
This detector scans for instances where a variable is casted to its own type. This is unnecessary and can be safely removed to improve code readability.

<details>

<summary>
There is 1 instance of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/libs/FixedMath.sol

25    return (n.toInt256() * FixedMath0x.FIXED_1) / int256(d.toInt256())
```
Unnecessary cast: `int256(d.toInt256())` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L25](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L25)


</details>

# 


## [G-06] Unused Named Return Variables
- Severity: Gas Optimization
- Confidence: High

### Description
Named return variables allow for clear and explicit naming of values to be returned from a function. However, when these variables are unused, it can lead to confusion and make the code less maintainable.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

174    function getPendingInterests(address user) external returns (PendingInterest[] memory pendingInterests) 
```
there is not use of this variables:
@ **pendingInterests**

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L194](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L174-L194)


#

```
File: contracts/Tokens/Prime/Prime.sol

496    function calculateAPR(address market, address user) external view returns (uint256 supplyAPR, uint256 borrowAPR) 
```
there is not use of this variables:
@ **supplyAPR**
@ **borrowAPR**

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496-L515](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L496-L515)


#

```
File: contracts/Tokens/Prime/Prime.sol

527    function estimateAPR(
528            address market,
529            address user,
530            uint256 borrow,
531            uint256 supply,
532            uint256 xvsStaked
533        ) external view returns (uint256 supplyAPR, uint256 borrowAPR) 
```
there is not use of this variables:
@ **supplyAPR**
@ **borrowAPR**

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L527-L548)


#

```
File: contracts/Tokens/Prime/libs/FixedMath.sol

53    function ln(int256 x) internal pure returns (int256 r) 
```
there is not use of this variables:
@ **r**

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L53-L55](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L53-L55)


#

```
File: contracts/Tokens/Prime/libs/FixedMath.sol

58    function exp(int256 x) internal pure returns (int256 r) 
```
there is not use of this variables:
@ **r**

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L58-L60](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/libs/FixedMath.sol#L58-L60)


</details>

# 



## [G-07] Inefficient Parameter Storage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 50

### Description
When passing function parameters, using the `calldata` area instead of `memory` can improve gas efficiency. Calldata is a read-only area where function arguments and external function calls' parameters are stored.

By using `calldata` for function parameters, you avoid unnecessary gas costs associated with copying data from `calldata` to memory. This is particularly beneficial when the parameter is read-only and doesn't require modification within the contract.

Using `calldata` for function parameters can help optimize gas usage, especially when making external function calls or when the parameter values are provided externally and don't need to be stored persistently within the contract.

<details>

<summary>
There is 1 instance of this issue:

</summary>

###
#

```
File: contracts/Tokens/Prime/Prime.sol

200    address[] memory users
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200](https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L200)


</details>

**[chechu (Venus) commented](https://github.com/code-423n4/2023-09-venus-findings/issues/357#issuecomment-1829646613):**

1. Acknowledged. It's an admin function and will be called only if values are different and needs an update. 

2. Acknowledged.

3. Acknowledged.

4. Fixed [here](https://github.com/VenusProtocol/venus-protocol/commit/5505bf7c2503f3228f3421b45ee749fce2c2921a).

5. Acknowledged.

6. Fixed [here](https://github.com/VenusProtocol/venus-protocol/commit/acfb5a7f8c8a1ba5a8b3d7b469558298eca61b6f).

7. Fixed [here](https://github.com/VenusProtocol/venus-protocol/commit/240f16f6cc53fa06a6389dfbb6fa821eb2b41b04).


***


# Audit Analysis

For this audit, 14 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-09-venus-findings/issues/465) by **3agle** received the top score from the judge.

*The following wardens also submitted reports: [J4X](https://github.com/code-423n4/2023-09-venus-findings/issues/455), [oakcobalt](https://github.com/code-423n4/2023-09-venus-findings/issues/243), [0x3b](https://github.com/code-423n4/2023-09-venus-findings/issues/195), [Breeje](https://github.com/code-423n4/2023-09-venus-findings/issues/170), [radev\_sw](https://github.com/code-423n4/2023-09-venus-findings/issues/660), [Bauchibred](https://github.com/code-423n4/2023-09-venus-findings/issues/646), [0xDetermination](https://github.com/code-423n4/2023-09-venus-findings/issues/577), [hunter\_w3b](https://github.com/code-423n4/2023-09-venus-findings/issues/563), [kaveyjoe](https://github.com/code-423n4/2023-09-venus-findings/issues/536), [ArmedGoose](https://github.com/code-423n4/2023-09-venus-findings/issues/413), [versiyonbir](https://github.com/code-423n4/2023-09-venus-findings/issues/350), [jamshed](https://github.com/code-423n4/2023-09-venus-findings/issues/262), and [0xweb3boy](https://github.com/code-423n4/2023-09-venus-findings/issues/241).*

## Index
- [Codebase Quality](#codebase-quality)
	- [Strengths](#strengths)
	- [Weaknesses](#weaknesses)
- [Mechanism Review](#mechanism-review)
	- [Architecture & Working](#architecture--working)
	- [Reward Mechanism](#reward-mechanism)
	- [Prime](#prime)
	- [Prime Liquidity Provider](#primeliquidityprovider)
- [Architecture Recommendations](#architecture-recommendations)
-  [Centralization Risks](#centralization-risks)
- [Systemic Risks](#systemic-risks)
- [Key Takeaways](#key-takeaways)

## Codebase quality

- The codebase exhibited remarkable quality. It has undergone multiple audits, ensuring the robustness of all major functionalities.

### Strengths

- Comprehensive unit tests
- Utilization of Natspec
- Adoption of governance for major changes, reducing centaralization risks.

### Weaknesses

- Documentation had room for improvement.

## Mechanism Review

### Architecture & Working

- Following are the main components of the Venus Prime system:
    1. **Prime.sol:**
        - *Description*: Prime.sol serves as the central contract responsible for distributing both revocable and irrevocable tokens to stakers.
        - *Functionality*: This contract plays a pivotal role in issuing Prime tokens to eligible stakers, enabling them to participate in the Venus Prime program and earn rewards.
    2. **ProtocolLiquidityProvider:**
        - *Description*: The ProtocolLiquidityProvider contract holds a reserve of funds that are designated for gradual allocation to the Prime contract over a predefined time period.
        - *Functionality*: This component ensures a consistent and planned provision of tokens to the Prime contract, guaranteeing a continuous supply of rewards for Prime token holders.
    3. **ProtocolShareReserve (OOS):**
        - *Description*: The ProtocolShareReserve acts as the repository for interest earnings generated across all prime markets within the Venus Protocol.
        - *Functionality*: It collects the interest earnings and acts as a centralized pool from which the Prime contract draws the necessary funds. These funds are then distributed as rewards to Prime token holders, contributing to the sustainability of the Venus Prime program.

![image](https://user-images.githubusercontent.com/131902879/283156338-96eb323b-87fc-4274-ae56-795f881689ed.png) 

### Rewarding Mechanism

![image](https://user-images.githubusercontent.com/131902879/283156465-4da087d6-902c-423e-b849-991f7105e453.png)

1. Initialization (Block 0):
    - Variables are set to default values.
2. Income and Scoring (Blocks 1-10):
    - Over 10 blocks, 20 USDT income is distributed.
    - User scores total 50, allocating 0.4 USDT per score point.
    - `markets[vToken].rewardIndex` increases by +0.4.
3. Income and Scoring (Blocks 11-30):
    - Next 20 blocks distribute 10 USDT.
    - User scores remain constant.
    - `markets[vToken].rewardIndex` increases by +0.2.
4. User Claims a Prime Token (Block 35):
    - User claims a Prime token.
    - `markets[vToken].rewardIndex` updated via `accrueInterest`.
    - Over 5 blocks, 5 USDT income with no score change.
    - `markets[vToken].rewardIndex` increases by +0.1.
    - `interests[market][account].rewardIndex` set to `markets[market].rewardIndex`.
5. User Claims Interest (Block 50):
    - User claims accrued interests.
    - `markets[vToken].rewardIndex` updated via `accrueInterest`.
    - Over 15 blocks, 120 USDT income with score change (sum of scores becomes 60).
    - `markets[vToken].rewardIndex` increases by +2.
6. Interest Calculation for the User:
    - User's interest calculated using `_interestAccrued`.
    - The difference between `markets[vToken].rewardIndex` (2.7) and `interests[market][account].rewardIndex` (0.7) is 2 USDT per score point.
    - Multiplied by user's score (10) equals 20 USDT accrued interest.
7. Updating User's Reward Index:
    - `interests[vToken][user].rewardIndex` set to the new `markets[vToken].rewardIndex`.

### Prime

![image](https://user-images.githubusercontent.com/131902879/283156537-fd5af46d-525b-43fc-bcd6-19ab175c17af.png)

- `Prime.sol` is a critical component of Venus Protocol, offering users the opportunity to earn rewards generated from specific markets within the protocol. Here's how it works:
    - To be eligible for a Prime token, regular users must stake a minimum of 1,000 XVS for a continuous period of 90 days.
    - Once this condition is met, users can claim their Prime token, signaling the beginning of rewards accrual.
    - Prime token holders have the flexibility to claim their accrued rewards at their convenience.

**Key Features and Functions in `Prime.sol`**:

- Utilizes the Cobb-Douglas function to calculate user rewards.
- Offers two types of Prime Tokens:
    - Revocable: Can be minted after staking a minimum of 1,000 XVS for 90 days but can be burnt if the stake falls below 1,000 XVS.
    - Irrevocable (Phase 2) - Specifics for this token type will be detailed in Phase 2 of the Venus Prime program.

### PrimeLiquidityProvider

![image](https://user-images.githubusercontent.com/131902879/283156622-93e889cc-40ee-4c95-9d37-11f9f704bcb9.png)

- `PrimeLiquidityProvider.sol` serves as the second source of tokens for the Prime program, complementing the tokens generated by Venus markets. Key details include:
    - This contract, `PrimeLiquidityProvider`, allocates a predetermined quantity of tokens to Prime holders in a uniform manner over a defined period.

**Key Features and Functions in `PrimeLiquidityProvider.sol`**:

- Ability to add new tokens for rewards accrual.
- Configurable token distribution speed.
- Safeguard mechanisms for accidentally sent tokens.
- Ability to pause and resume token transfers, ensuring control and stability within the program.

## Architecture Recommendations

- **Staking and Earning Venus Prime Tokens**: These are essential parts of the Venus Prime Yield Protocol. Let's break them down and talk about ways to make them even better:
- **Staking as it is now**:
    - Right now, if you want to earn a Prime Token, you have to lock up at least 1,000 XVS for 90 days. This Prime Token represents the XVS you've staked and comes with some perks. But if you take out some XVS and drop below 1,000, you'll lose your Prime Token.
- **Ways to Make Staking Better**:
    - **Different Staking Levels**: Instead of one fixed requirement, we could have different levels for staking. This way, even if you have fewer tokens, you can join in. Different levels might offer different bonuses to encourage more staking.
    - **Flexible Staking Time**: We could let you choose how long you want to stake your XVS, depending on what suits you. Staking for longer could mean more rewards, which could motivate people to keep their XVS locked up longer.
    - **Smart Staking Rules**: The rules for how much you need to stake could change depending on how the market is doing and how much XVS is worth. This could help keep things steady and reliable.
- **What You Can Do with Venus Prime Tokens**:
    - **More Ways to Use Them**: We could give you more things to do with your Prime Tokens, so they're even more useful in the Venus ecosystem. That could encourage more people to join in.
    - These tokens are like keys to the Venus ecosystem. You can use them to vote on changes, earn rewards, and access different parts of the system.
- These changes could make staking easier and more rewarding, and make Venus Prime Tokens more valuable. It's all about making the system better for everyone.

## Centralization Risks

- **Smart Contract Centralization**: The protocol is based on smart contracts, which are autonomous and decentralized by nature. However, the developers of the protocol have the ability to update or modify these contracts, introducing a potential point of centralization. If these powers are abused or misused, it could lead to centralization risks
- **Governance Centralization**: The governance of Venus Prime is controlled by XVS token holders. If a small group of individuals or entities comes to own a large portion of XVS tokens, they could potentially control the governance of the protocol, leading to centralization. This can include making decisions that favor them at the expense of other users

## Systemic Risks

1. **Governance Risk**: The protocol faced a hacking attempt where the attacker tried to gain control of the protocol by bribery (VIP42). Although the attempt was thwarted, it highlights the risk of governance attacks
2. **Risk Fund Adequacy**: Venus has a risk fund established to address potential shortfalls in the protocol, particularly in situations of ineffective or delayed liquidations. However, if the fund is not adequate to cover a major event, it could potentially result in a systemic risk
3. **Price Oracle Resilience**: Venus V4 introduces resilient price feeds. If these feeds fail or provide inaccurate data, it could potentially destabilize the system. The protocol's resilience to such an event is yet to be tested
4. **Dependence on XVS Staking**: Venus Prime requires users to stake at least 1,000 XVS for 90 days to mint their Prime Token. If a user decides to withdraw XVS and their balance falls below 1,000, their Prime Token will be automatically revoked. This introduces a risk if there's a significant drop in the value of XVS or if a large number of users decide to unstake and sell their XVS simultaneously

## Key Takeaways

1. **Dual Token System:** The protocol introduces a bifurcated token system, comprising revocable and irrevocable Prime tokens. Each token variant carries its unique rules and benefits, offering users a versatile array of choices.
2. **Sustainable Rewards:** Diverging from conventional incentive models dependent on external sources, Venus Prime generates its rewards intrinsically within the protocol. This inherent mechanism not only fosters sustainability but also augments the potential for long-term growth.
3. **Long-Term Commitment:** Users are incentivized to uphold a commitment to the protocol by staking XVS tokens for a minimum duration of 90 days. This prolonged engagement serves the dual purpose of fostering dedication and dissuading premature withdrawals.
4. **Complex Reward Calculation:** Venus Prime employs a sophisticated reward calculation formula known as the Cobb-Douglas function to ascertain user rewards. This intricacy, while daunting on the surface, is intricately designed to uphold principles of equity and precision in reward distribution.

### Time spent:
45 hours

***
# Disclosures

C4 is an open organization governed by participants in the community.

C4 Audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
