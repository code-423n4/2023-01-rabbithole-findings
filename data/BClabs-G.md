# [01] Comparing bool with bool is redundant

```
        return claimedList[tokenId_] == true;
```

No need for comparison, just return the bool.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L136

# [02] Save on for loops

getOwnedTokenIdsOfQuest could be written using EnumerableSet instead of 2 for loops. Instead of saving the return values in one array and then filtering it, one could simply save the return values in an uint EnumerableSet and use values to retrieve all the values saved. It should however be implemented using memory variables instead of saving and reading from storage.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L109-L135