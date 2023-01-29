G1 Avoid keccak256 calls and use string compare 
quest-protocol/contracts/QuestFactory.sol line 105
if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155')))