## 1. Sweep Token Will Emit Incorrect Event With Fee-On-Transfer Tokens

Description:

The sweep token function here https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L216 when given a fee-on-trasfer token (considering there is some amount of that token in the contract) will emit the incorrect event as the amount in the argument of the event  would be less than the amount given as parameter.


## 2. getEffectivedistributionSpeed Might Give Incorrect Result

Description:

The function getEffectivedistributionSpeed here https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L232 might give incorrect result as before calculating `uint256 accrued
` it does not call accrueTokens , since it is a view function is would be incorrect to change state this way but it can be changed to a normal external function.

## 3. While Issuing  a Prime Token _mint Might Revert

Description:

The appropriate role can call issue() function https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L331 to issue a Prime token to an array of users  , but if one of the user already has a prime token the whole call will revert due to the checks in the _mint function that if the user already has a prime token then revert.
Ensure that before minting the user does not have a prime token.

## 4 . _accrueInterestAndUpdateScore Should Be Used

We accrue interest and update score of the users across all markets in this code block https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L211-L217  , but instead of this a simple call to `_accrueInterestAndUpdateScore` would be sufficient since it does the same job and would make the code more readable and streamlined.

## 5 . isScoreUpdated Might Not Be Updated For A Round Id

Description:

updateAlpha , updateMultiplers , addMarket when called updates a round Id through _startScoreUpdate round at L818 and score updates for these rounds take place in the function updateScores() at L200 . If for a round before updateScores() is called for a user the next round gets started ( through updateAlpha , updateMultiplers , addMarket ) then score won’t be updated for that round for the users and isScoreUpdated  will always be false for that round.

## 6 . User Might Mint a Prime Token Even Though The Stake Period Was Less Than 90 Days

Description:

When the user has staked enough XVS he can call xvsUpdated() to set his `stakedAt` mapping to the current timestamp . Then the user has to wait for 90 days to claim his prime token while the minimum required XVS is staked , but the user might unstake in between and call claim() at L397 after 90 days with insufficient stake and still have a prime token minted. This will be serious if there are less users in the system which would not guarantee the liquidation process through xvsUpdated.


## 7 . Capital Calculation Might Revert In Future Upgrades

Description:

Capital is calculated here https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L661 , currently vToken will be 8 decimals but other are possible in future , if in future the decimal is more than 18 , capital calculation will always revert and no scores calculated.

## 8 . Incorrect Comment

Description:

The comment here https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/PrimeLiquidityProvider.sol#L23 is incorrect , it should be “ the last block where tokens were accrued” instead of “The rate at which token is distributed to the Prime contract” 


