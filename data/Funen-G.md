1. Reorder code can save more gas deployment

Since :
```
uint256 is 32 bytes
uint128 is 16 bytes
uint64 is 8 bytes
address (and address payable) is 20 bytes
bool is 1 byte
string is usually one byte per character
```

[this line](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L13-L22) can save gas opt by doing this :

#BEFORE

```
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

```
// 140851 gas  before 


#AFTER

```
   
    uint256 public immutable startTime;
    uint256 public immutable endTime;
    uint256 public immutable totalParticipants;
    uint256 public immutable rewardAmountInWeiOrTokenId;
    address public immutable rewardToken;

    uint256 public redeemedTokens;

    bool public hasStarted;
    bool public isPaused;

    string public questId;


    RabbitHoleReceipt public immutable rabbitHoleReceiptContract;

```

// 140195 gas after