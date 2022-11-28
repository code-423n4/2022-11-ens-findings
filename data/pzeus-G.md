Use `calldata` instead of `memory` in order to save some gas fees
https://github.com/code-423n4/2022-11-redactedcartel/blob/main/src/PirexGmx.sol#L863

Change `string memory _delegationSpace` to `string calldata _delegationSpace`