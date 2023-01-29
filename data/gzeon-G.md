## Struct packing

Some struct can be packed tighter to save storage slot, for example:

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L19-L24
```
    struct Quest {
        mapping(address => bool) addressMinted;
        address questAddress;
        uint totalParticipants;
        uint numberMinted;
    }
```

can be packed into

```
    struct Quest {
        mapping(address => bool) addressMinted;
        address questAddress;
        uint48 totalParticipants;
        uint48 numberMinted;
    }
```

To save 2 storage slot, while still allow 2^48 participant and reward to be minted which should be reasonably enough.

## Deploy with proxy

Quest contracts are deployed directly from the factory, which can be expensive. Consider deploying proxy to a shared Quest implementation to reduce deployment gas cost. Although this will increase each call to the contract slightly.

## Redundant hash parameter

hash_ is redundant since it can be calculated with other input

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219-L222
```
    function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
        if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
        if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
```

## Inline recoverSigner into hash calculation

Instead of abi.encodePacked twice, and wasting gas copying the data into memory twice, 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L210-L213
```
    function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) {
        bytes32 messageDigest = keccak256(abi.encodePacked('\x19Ethereum Signed Message:\n32', hash_));
        return ECDSAUpgradeable.recover(messageDigest, signature_);
    }
```

hash it here altogether, the actual message signed would need to be slightly changed (instead of signing a typed hash, sign the typed message)

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L222
```
        if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
```