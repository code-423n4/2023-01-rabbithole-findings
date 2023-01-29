### Cache keccak256(abi.encodePacked(contractType_)) in QuestFactory
By caching  `keccak256(abi.encodePacked(contractType_))` you can save **5.397gas** in deployment and **137gas** in function calling.
The above results are by calling yarn test:gas-stories and applying following changes:

#L71 new statement: `bytes32 contractTypeCached = keccak256(abi.encodePacked(contractType_));`
#L72 change `if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20')))` to `if (contractTypeCached == keccak256(abi.encodePacked('erc20')))`
#L105 change `if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155')))` to `if (contractTypeCached == keccak256(abi.encodePacked('erc1155')))`

### Change pause function in contracts/Quest.sol
Save gas by changing the two functions `pause()` and `unpause()` with the following function:
    
    function pause(bool pause_) public onlyOwner onlyStarted {isPaused = pause_;}

Running  yarn test:gas-stories there's **40.897gas** saving in QuestFactory deployment, **19.419gas** in calling createQuest function and a smaller save in claiming all other functions belonging to Erc1155Quest and Erc20Quest contracts.

### Cache keccak256(bytes(questId_)) outside for loop in contracts/RabbitHoleReceipt.sol#L119
By caching  `keccak256(bytes(questId_))` outside for loop you can save **696gas** in deployment of RabbitHoleReceipt.
If there's only 1 token to be claimed the claim() function call increases by **8gas** but if they are more a save is expected.
The above results are by calling yarn test:gas-stories and applying following changes:

#L116 new statement: `bytes32 tempQuestId = keccak256(bytes(questId_));`
#L119 change `if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_)))` to `if (keccak256(bytes(questIdForTokenId[tokenId])) ==  tempQuestId )`
