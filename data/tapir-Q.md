1-
Assume the vUSDC market has not in sync for 8 hours and there are some interest waiting to accrue when someone interacts with the comptroller. Alice who has both capital on the vUSDC market and the XVS in the vault increases its XVS vault holdings. As a result of this tx from Alice Alice's score will be recalculated with the new XVS amount. However, these lines are taking the latest values from the Comptroller without factoring the current interest that needs to be accrued:
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L651C1-L652

If the Comptrollers accrue interest would've called before Alice interacted with the XVS, Alice's score would be higher because the exchangeRate would be increased due to pending interests. In this case, Alice needs to send an another tx to update her score which will lead her an extra tx.

Also this scenario is valid for all the users that has a score. When a user say Bob has a score of X and he interacted with the protocol last time 2 months ago can call "accrueInterestAndUpdateScore" to increase its score significantly since the 2 months is a long time interval and in this interval the exchangeRate is definitely higher than the initially when Bob interacted with the prime token contracts. This means that the users needs to call "accrueInterestAndUpdateScore" often enough to get the more score, or simply interact with the Comptroller. 

This does not look like a medium issue so I will label this as Low/Informational