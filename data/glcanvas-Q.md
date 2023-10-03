# [L-01] Storage sometimes doesn't cleared correctly 
 
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L340-L341
https://github.com/code-423n4/2023-09-venus/blob/main/contracts/Tokens/Prime/Prime.sol#L378
Possible to become Irrevocable user and doesn't clear `stackedAt` correctly

Steps to reproduce:

1) User stacked 1000 XVS tokens, `stakedAt[user] = block.timestamp;` filled;
2) Admin calls `issue` with user and, intends to make this user irrevocable;
3) Code flow will be executed here: ```                    _mint(true, users[i]);
                    _initializeMarkets(users[i]); ```, because user not exist (in staking phase)
4) Here user becomes irrevocable, but he still in stake phase and from now field `stakedAt[user]` will be non zero; 

#[L-02] Fix Typos 
`accrues interes and updates score for an user for a specific market` -> `interest`
`tokens_ Array of addresses of the tokens to be intialized` -> `initialized` 
`mapping of asset adress => amount` -> `address`

