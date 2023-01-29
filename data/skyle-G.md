G1 Avoid keccak256 calls and use string compare 
quest-protocol/contracts/QuestFactory.sol line 105
if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155')))

G2 calculate keccak256 out side of the for loop to save gas
quest-protocol/contracts/RabbitHoleReceipt.sol line 117
for (uint i = 0; i < msgSenderBalance; i++) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
            }
        }
change to
hash =  keccak256(bytes(questId_))
for (uint i = 0; i < msgSenderBalance; i++) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == hash) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
            }
        }