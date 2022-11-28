# QA

Label | Title
---|---
L00 | NameWrapper: Missing `isWrapped` function
L01 | NameWrapper: `upgrade` does not revert when called with ETH2LD

## LOW

### L00 NameWrapper: Missing `isWrapped` function

According to the `wrapper/README.md`:

> To check if a name has been wrapped, call `isWrapped()`. This checks:
> 
> - The NameWrapper is the owner in the Registry contract
> - The owner in the NameWrapper is non-zero

However, there is no implementation of the `isWrapped()` function.


### L01 NameWrapper: `upgrade` does not revert when called with ETH2LD

The `NameWrapper.upgrade` function is supposed to be called only by non .eth domain, based on the comment. However, it currently lacks the check whether the given `parentNode` is not the `ETH_NODE`, and it allows to be called by .eth node as the proof of concept shows.

This is, however, reported as QA, assuming the upgraded NameWrapper has some logic to check the parentNode is not `ETH_NODE`.
Nevertheless, to ensure that no .eth node can be called with `NameWrapper.upgrade`, it is probably good to have the check on the current NameWrapper.

https://github.com/code-423n4/2022-11-ens/blob/2b0491fee2944f5543e862b1e5d223c9a3701554/contracts/wrapper/NameWrapper.sol#L436

```solidity
// NameWrapper.sol
 426     /**
 427      * @notice Upgrades a non .eth domain of any kind. Could be a DNSSEC name vitalik.xyz or a subdomain
 428      * @dev Can be called by the owner or an authorised caller
 429      * Requires upgraded Namewrapper to permit old Namewrapper to call `setSubnodeRecord` for all names
 430      * @param parentNode Namehash of the parent name
 431      * @param label Label as a string of the name to upgrade
 432      * @param wrappedOwner Owner of the name in this contract
 433      * @param resolver Resolver contract for this name
 434      */
 435
 436     function upgrade(
 437         bytes32 parentNode,
 438         string calldata label,
 439         address wrappedOwner,
 440         address resolver
 441     ) public {
 442         bytes32 labelhash = keccak256(bytes(label));
 443         bytes32 node = _makeNode(parentNode, labelhash);
 444         (uint32 fuses, uint64 expiry) = _prepareUpgrade(node);
 445         upgradeContract.setSubnodeRecord( 
 446             parentNode,
 447             label,
 448             wrappedOwner,
 449             resolver,
 450             0,
 451             fuses,
 452             expiry
 453         );
 454     }
```

```solidity
// Proof of concept
// to run: add this code to the https://gist.github.com/zzzitron/7670730176e35d7b2322bc1f4b9737f0#file-2022-11-ens-versus-poc-t-sol-L345

    function testTest2() public {
        // using the mock for upgrade contract
        deployNameWrapperUpgrade();
        string memory node_str = 'vitalik.eth';
        string memory sub1_full = 'sub1.vitalik.eth';
        string memory sub1_str = 'sub1';
        (, bytes32 node) = node_str.dnsEncodeName();
        (bytes memory sub1_dnsname, bytes32 sub1_node) = sub1_full.dnsEncodeName();

        // wrap parent and lock
        vm.prank(user1);
        registrar.setApprovalForAll(address(nameWrapper), true);
        vm.prank(user1);
        nameWrapper.wrapETH2LD('vitalik', user1, type(uint16).max /* all fuses are burned */, address(0));
        // sanity check
        (address owner, uint32 fuses, uint64 expiry) = nameWrapper.getData(uint256(node));
        assertEq(owner, user1);
        assertEq(fuses, PARENT_CANNOT_CONTROL | IS_DOT_ETH | type(uint16).max);
        assertEq(expiry, 2038123728);

        // upgrade as nameWrapper's owner
        vm.prank(root_owner);
        nameWrapper.setUpgradeContract(nameWrapperUpgrade);
        assertEq(address(nameWrapper.upgradeContract()), address(nameWrapperUpgrade));

        // user1 calls upgradeETH2LD
        vm.prank(user1);
        // nameWrapper.upgradeETH2LD('vitalik', address(123) /* new owner */, address(531) /* resolver */);
        // The line below does not revert unless the upgraded NameWrapper has the logic to check the parentNode
        nameWrapper.upgrade(ETH_NODE, 'vitalik', address(123) /* new owner */, address(531) /* resolver */);
    }
```

