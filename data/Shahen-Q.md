## Zero-Address check for calculateAPR() function.

### Summary;
The calculateAPR() function takes two address arguments 'market' and 'user'. It does not check that the address are valid.Therefore requires a zero address check.

### Code Snippet;
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L496

## Zero-Address check for estimateAPR() function.

### Summary;
The estimateAPR() function's address arguments 'market' and 'user'. It does not check that the address are valid.Therefore requires a zero address check.

### Code Snippet;
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L527



