# 1. remove mintReceipt function's hash_ parameter

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219

As the hash parameter can only equal to `keccak256(abi.encodePacked(msg.sender, questId_))`, we can delete this parameter.

Suggestion: change to:

```
    function mintReceipt(string memory questId_, bytes memory signature_) public {
        if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
        bytes32 hash_ = abi.encodePacked(msg.sender, questId_);
        if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();

        quests[questId_].addressMinted[msg.sender] = true;
        quests[questId_].numberMinted++;
        emit ReceiptMinted(msg.sender, questId_);
        rabbitholeReceiptContract.mint(msg.sender, questId_);
    }
```

# 2. QuestFactory's quests can be marked as private

quests variable is public, but do not give more information than `questInfo` function. (solc compiler do not support generating query function for a mapping inside of the struct)

So marking variable `quests` as private will save gas, or remove questInfo function.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L28

```
    struct Quest {
        mapping(address => bool) addressMinted;
        address questAddress;
        uint totalParticipants;
        uint numberMinted;
    }

mapping(string => Quest) public quests;

    /// @dev return data in the quest struct for a questId
    /// @param questId_ The id of the quest
    function questInfo(string memory questId_) external view returns (address, uint, uint) {
        return (
            quests[questId_].questAddress,
            quests[questId_].totalParticipants,
            quests[questId_].numberMinted
        );
    }
```