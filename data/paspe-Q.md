#### [L-01] 
## Unlicensed contracts
#### Summary
Some of the files [QuestFactory, Quest, Erc20Quest, Erc1155Quest, IQuest, IQuestFactory] have Unlicensed keyword. Better is to have MIT license 
#### Github permalinks
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L1
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L1
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L1
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L1
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L1
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L1

#### [NC-02]
## Not last version OpenZeppelin package.json
#### Summary
In all listed contracts for audit it is used OpenZeppelin library 4.8.0 but in the setup of the project `package.json` it is not used the last stable version.
It needs to be used the latest stable version 4.8.1 to have more recent and protected version with latest fixes.
#### Github permalinks
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/package.json#L41-L42 
