In `Prime.sol` 
https://github.com/code-423n4/2023-09-venus/blob/b11d9ef9db8237678567e66759003138f2368d23/contracts/Tokens/Prime/Prime.sol#L316-L324

The `emit` operation is executed prior to the assignment of new values.

Suggestion
Do 2 `emit`s 
One before assignment with the old values
Then assignment.
Another `emit` event that shows current values
```
...
emit MintLimitsUpdated(irrevocableLimit, revocableLimit);
revocableLimit = _revocableLimit;
irrevocableLimit = _irrevocableLimit;
emit MintLimitsUpdated(irrevocableLimit, revocableLimit);
}
```
