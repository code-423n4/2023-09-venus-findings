https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L443-L446

Allowing anyone to claim on behalf for other users might be 
annoying to the actual token owner

The current contract implementation allows other users to call `claimInterest` on other users behalf, This might be annoying to users if they want to keep earning interest on the soulbound tokens but another  user calls the `claimInterest` on their behalf leading to early withdrawal of interest.

```
function claimInterest(address vToken, address user) external whenNotPaused returns (uint256) {
```
It is advisable not to allow other users call `claimInterest` on behalf of other user Or implement an approval mechanism.