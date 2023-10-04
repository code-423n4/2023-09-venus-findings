# User without a prime token can get a value for interests[vToken][user].rewardIndex set

Since any user can call `claimInterest(address, address)`, the value of `interests[vToken][user].rewardIndex`
 can be set to a non-zero value even if the user doesn't have a prime token.

[Code that sets the interests[vToken][user].rewardIndex for any user.](https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L676)

## POC
To run the POC, go to `tests/hardhat/Prime/Prime.ts` add an `attacker` signer, and paste the `it` block below inside `describe("boosted yield", () => {...})`.

```solidity
it.only("User without a prime token can get a value for interests[vToken][user].rewardIndex set", async () => {
      // Mock so markets[vToken].rewardIndex != 0
      await protocolShareReserve.getUnreleasedFunds.returns("518436");
      await prime.accrueInterest(vusdt.address);

      expect((await prime.interests(vusdt.address, attacker.getAddress())).rewardIndex).to.eq(0);
      await prime.connect(attacker)["claimInterest(address,address)"](vusdt.address, attacker.getAddress());
      expect((await prime.interests(vusdt.address, attacker.getAddress())).rewardIndex).to.be.greaterThan(0);
    });
```

## Recommended Mitigation Steps
`_claimInterest(address, address)` should check whether the user has a prime token. If not, don't update interests[vToken][user].rewardIndex as it should already be 0 from when the token was burnt.