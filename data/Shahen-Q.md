## Zero-Address check for calculateAPR() function.

### Summary;
In line 496, The calculateAPR() function takes two address arguments 'market' and 'user'. It does not check that the address are valid.

### Code Snippet;
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L496

