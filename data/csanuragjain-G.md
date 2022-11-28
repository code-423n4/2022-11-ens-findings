## No need of _doSafeBatchTransferAcceptanceCheck when ids.length == amounts.length=0

Contract:
https://github.com/code-423n4/2022-11-ens/blob/main/contracts/wrapper/ERC1155Fuse.sol#L202

Issue:
In safeBatchTransferFrom function, If both ids and amount length is 0 then there is practically nothing to transfer. This means there is no need to make call to `_doSafeBatchTransferAcceptanceCheck` as nothing is actually transferred

Recommendation:
Return early when ids.length == amounts.length=0

```
if(ids.length==0) return;
```

__ 

## No need of expiry check

Contract:
https://github.com/code-423n4/2022-11-ens/blob/main/contracts/wrapper/ERC1155Fuse.sol#L247

Issue:
In _mint function, Since expiry can always increase and never actually decrease so there is no need to check whether oldExpiry > expiry. The same check can be skipped

Recommendation:
Remove the check

```diff
- if (oldExpiry > expiry) {
-            expiry = oldExpiry;
 -       }
```