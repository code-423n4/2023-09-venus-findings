https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L129-L144


In the PrimeLiquidityProvider contract there is no check to ensure that fundstransfer is not already paused so a privileged user can call the pauseFundsTransfer() function even though it is already paused thereby wasting gas.
Same thing in resumeFundsTransfer()
Add a require statement to ensure that these functions are not already active. 