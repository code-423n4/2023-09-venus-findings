## Line 200 (Function argument), If the 'users' array becomes excessively large, it can potentially lead to excessive gas consumption. Recommend using calldata instead of memory to reduce the consumption of gas.

### proof of concept

    `function updateScores(address[] memory users) external { // original function
        
    }`

    `function updateScores2(address[] calldata users) external { // gas-Optimized function
       
    }`


   //Foundry Gas report (Console output) of the two functions using 5 addresses as test input.

  Function Name                      | min           | avg  | median | max  | # calls |
  | updateScores                     | 1266          | 1266 | 1266   | 1266 | 1       |
  | updateScores2                    | 397           | 397  | 397    | 397  | 1       |

### Code Snippet;
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L200