### createQuest function doesn't check all arguments
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73
It's suggested to carry out a sanity check of parameters before creating new quests.
There are parameters that can impair the new quest, wasting gas and requiring to create a new quest with a different `questId_` if are inputed not correctly.
Time related arguments are checked in Quest.sol cosntructor, but it is suggested to check also `totalParticipants_ != 0` `rewardAmountOrTokenId_ != 0` and that addresses are not =0.



### Parent contract ERC721EnumerableUpgradeable not initialized   
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L49
Parent contract ERC721EnumerableUpgradeable is not initialized as done per ERC721Upgradeable and ERC721URIStorageUpgradeable.
This should not cause great issues because initializing doesn't perform anything in this case but it's a good practice to call it. 



### Unused function parameters
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L122
`redeemableTokenCount_` is not used in `function _calculateRewards(uint256 redeemableTokenCount_)`



https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L129
`amount_` is not used in `function _transferRewards(uint256 amount_)`



### Use a struct for mapping
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L30
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L34
In order to keep code neat it is suggeste to substitute `questIdForTokenId` and `timestampForTokenId` with the following:

    struct InfoForTokenId{
        string questId;
        uint256 timestamp;
     }
     mapping(uint => InfoForTokenId) public infoForTokenId;



### function royaltyInfo() require uint256 tokenId without using it
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L179
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L110
In both functions the parameter uint256 tokenId is required but without real use: in RabbitHoleReceipt implementation there's only  a check that `require(_exists(tokenId_), 'Nonexistent token');` while in RabbitHoleTickets is completely unused.
