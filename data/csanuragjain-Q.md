## No need of safeBatchTransferFrom function

Contract:
https://github.com/code-423n4/2022-11-ens/blob/main/contracts/wrapper/ERC1155Fuse.sol#L202

Issue:
Both functions `_doSafeBatchTransferAcceptanceCheck` and `_doSafeTransferAcceptanceCheck` are doing same job, one for multiple id and other for single id. We can combine both by revising the `safeBatchTransferFrom` function

Recommendation:
Revise the `safeBatchTransferFrom` function

```
function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts,
        bytes memory data
    ) public virtual override {
...
for (uint256 i = 0; i < ids.length; ++i) {
            uint256 id = ids[i];
            uint256 amount = amounts[i];

            (address oldOwner, uint32 fuses, uint64 expiry) = getData(id);

            _preTransferCheck(id, fuses, expiry);

            require(
                amount == 1 && oldOwner == from,
                "ERC1155: insufficient balance for transfer"
            );
            _setData(id, to, fuses, expiry);

_doSafeTransferAcceptanceCheck(msg.sender, from, to, id, amount, data);

        }

emit TransferBatch(msg.sender, from, to, ids, amounts);
}
```