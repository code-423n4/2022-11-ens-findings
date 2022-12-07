### 1. Redundant code in `setSubnodeOwner` and `setSubnodeRecord`

Specifically, `setSubnodeRecord` invokes `_storeNameAndWrap` and `_updateName`, and `setSubnodeOwner` invokes `_updateName`, to update the `names[]` db.

These part of updating `names[]` could be removed (and hence save gas), because both functions will inovke `_saveLabel` to update `names[]`

### 2. Inconsistant between `unwrap` and `setRecord`

Basically, for function [unwrap](https://github.com/ensdomains/ens-contracts/blob/bc6ff56de5f15435feff4825085d53ae689ebf15/contracts/wrapper/NameWrapper.sol#L373), we are not able to set the owner in the registry as 0. 

But in function [setRecord](https://github.com/ensdomains/ens-contracts/blob/bc6ff56de5f15435feff4825085d53ae689ebf15/contracts/wrapper/NameWrapper.sol#L613-L617), we can do so.

### 3. ETH2LD node can be upgraded via `NameWrapper::upgrade`

Basically, unlike other functions, `NameWrapper::upgrade` function does not check whether the subject node is an ETH2LD node, and can migrate the node accordingly. However, such a procedure only transfers the `ens.owner` rather than the registrar ERC721 token.

```js
    it('Attack via upgrade an ETH2LD node', async () => {
      await NameWrapper.setUpgradeContract(NameWrapperUpgraded.address)

      await BaseRegistrar.register(labelhash('sub1'), hacker, 1 * DAY)

      await BaseRegistrarH.setApprovalForAll(NameWrapper.address, true)
      await NameWrapperH.wrapETH2LD('sub1', hacker, 0, DUMMY_ADDRESS)

      await NameWrapperH.upgrade(namehash('eth'), 'sub1', hacker, EMPTY_ADDRESS)

      // + only the owner of ENS registry is in the new NameWrapper
      // + the BaseRegistrar ERC721 is owned by the old NameWrapper
      expect(await EnsRegistry.owner(namehash('sub1.eth'))).to.equal(NameWrapperUpgraded.address)
      expect(await BaseRegistrar.ownerOf(labelhash('sub1'))).to.equal(NameWrapper.address)
    })
```
