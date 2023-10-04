## _burn can be adusted to improve readability and avoid unnecessary storage remodification when not to have done
change:
```solidity
        if (tokens[user].isIrrevocable) {
            totalIrrevocable--;
        } else {
            totalRevocable--;
        }

        tokens[user].exists = false;
        tokens[user].isIrrevocable = false;
```
to this:
```diff
        if (tokens[user].isIrrevocable) {
            totalIrrevocable--;
+           tokens[user].isIrrevocable = false;
        } else {
            totalRevocable--;
        }

        tokens[user].exists = false;
-       tokens[user].isIrrevocable = false;
```