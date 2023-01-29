# There is room for sparing gas by taking out one unecessary `for` on the `RabbitHoleReceipt.getOwnedTokenIdsOfQuest` function

The function `RabbitHoleReceipt.getOwnedTokenIdsOfQuest` uses one `for` to get all token ids owned by `claimingAddress_`. Then it uses another `for` to validate (or filter) those tokens and returns them in the `filteredTokens` variable. But the filtering can be done on the first `for`, therefore sparing gas.

The function below can be replaced by a new one (se the new function suggestion after the code below): `RabbitHoleReceipt.getOwnedTokenIdsOfQuest`
```solidity
    function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory) {
        uint msgSenderBalance = balanceOf(claimingAddress_);
        uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
        uint foundTokens = 0;

        for (uint i = 0; i < msgSenderBalance; i++) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
            }
        }

        uint[] memory filteredTokens = new uint[](foundTokens);
        uint filterTokensIndexTracker = 0;

        for (uint i = 0; i < msgSenderBalance; i++) {
            if (tokenIdsForQuest[i] > 0) {
                filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
                filterTokensIndexTracker++;
            }
        }
        return filteredTokens;
    }
```
The new function suggestion, using only one `for`: `RabbitHoleReceipt.getOwnedTokenIdsOfQuest`
```solidity
    function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory,uint numberOfFoundTokens) {
        uint msgSenderBalance = balanceOf(claimingAddress_);
        uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
        uint foundTokens = 0;
        uint[] memory filteredTokens = new uint[](msgSenderBalance);
        uint filterTokensIndexTracker = 0;

        for (uint i = 0; i < msgSenderBalance; i++) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
                if(tokenId>0){
                    filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
                    ++filterTokensIndexTracker;
                }
            }
        }
        return (filteredTokens,filterTokensIndexTracker);
    }
```

Also the function `Quest.claim` can be changed from: 
```solidity
    function claim() public virtual onlyQuestActive {
        ...
        uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);

        if (tokens.length == 0) revert NoTokensToClaim();

        uint256 redeemableTokenCount = 0;
        for (uint i = 0; i < tokens.length; i++) {
            if (!isClaimed(tokens[i])) {
                redeemableTokenCount++;
            }
        }
        ...
    }
```
to:

```solidity
    function claim() public virtual onlyQuestActive {
        ...
        (uint[] memory tokens,uint numberOfTokens) = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);

        if (numberOfTokens == 0) revert NoTokensToClaim();

        uint256 redeemableTokenCount = 0;
        for (uint i = 0; i < numberOfTokens; i++) {
            if (!isClaimed(tokens[i])) {
                redeemableTokenCount++;
            }
        }
        ...
    }
```