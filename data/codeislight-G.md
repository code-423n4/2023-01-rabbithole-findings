- Quest struct can be optimized to fit questAddress, totalParticipants, and numberMinted in 1 storage slot, to result in the following:
```
    struct Quest {
        mapping(address => bool) addressMinted;
        address questAddress;
        uint48 totalParticipants;
        uint48 numberMinted;
    }
```
- use contractType enum instead of string memory, to help reduce any ambiguity and save gas on keccak256 and abi.encodePacked. as an example the following enum can be used:
```
enum TOKEN_TYPE {ERC20,ERC1155};
``` 
- questId variable value doesn't change in Quest contract, advised to be defined as immutable to save on gas.  
- in QuestFactory.mintReceipt, the following check statement:
```
        if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
```
equates to:
```
        if (quests[questId_].numberMinted == quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
```
which would save on 3 gas on extra add opcode.

- no need to double check for boolean value, it is sufficient by its own, and it would save 24 gas:
```
        if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();

        return claimedList[tokenId_] == true;

```
equates to:
```
        if (!rewardAllowlist[rewardTokenAddress_]) revert RewardNotAllowed();

        if (quests[questId_].addressMinted[msg.sender]) revert AddressAlreadyMinted();

        return claimedList[tokenId_];
```