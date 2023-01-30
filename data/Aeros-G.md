### Gas Optimizations:

The state storage variables in [**Quest.sol**](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol) can be rearranged in a different order to save gas by packing smaller than 32 bytes variables together.

Original:

    RabbitHoleReceipt public immutable rabbitHoleReceiptContract;
    address public immutable rewardToken;
    uint256 public immutable endTime;
    uint256 public immutable startTime;
    uint256 public immutable totalParticipants;
    uint256 public immutable rewardAmountInWeiOrTokenId;
    bool public hasStarted;
    bool public isPaused;
    string public questId;
    uint256 public redeemedTokens;

Possible adjustment:

    RabbitHoleReceipt public immutable rabbitHoleReceiptContract;
    bool public hasStarted;
    bool public isPaused; // (packs bool vars in the same slot as the RabbitHoleReceipt address)
    address public immutable rewardToken;
    uint256 public immutable endTime;
    uint256 public immutable startTime;
    uint256 public immutable totalParticipants;
    uint256 public immutable rewardAmountInWeiOrTokenId;
    string public questId;
    uint256 public redeemedTokens;

The following lines of code contain a comparison to a boolean constant, which is unnecessary:

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L136
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221
