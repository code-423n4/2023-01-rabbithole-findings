## [G-1] state variables can be packed in fewer slots
Here is the permalink of it [Quest.sol#L13-L22](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L13-L22)

`    RabbitHoleReceipt public immutable rabbitHoleReceiptContract;`
 `   address public immutable rewardToken;`
  `  uint256 public immutable endTime;`
  `  uint256 public immutable startTime;`
 `   uint256 public immutable totalParticipants;`
   ` uint256 public immutable rewardAmountInWeiOrTokenId;`
   ` bool public hasStarted;`
   ` bool public isPaused;`
    `string public questId;`
  `  uint256 public redeemedTokens;`

here  `  address public immutable rewardToken;` which occupies 20 bytes which can easily clubbed with   ` bool public hasStarted;` (1 byte) &  ` bool public isPaused;` (1 byte) this can save 1 S slot 

after changes 

`    RabbitHoleReceipt public immutable rabbitHoleReceiptContract;`
  `  uint256 public immutable endTime;`
  `  uint256 public immutable startTime;`
 `   uint256 public immutable totalParticipants;`
   ` uint256 public immutable rewardAmountInWeiOrTokenId;`
 `  address public immutable rewardToken;`
   ` bool public hasStarted;`
   ` bool public isPaused;`
    `string public questId;`
  `  uint256 public redeemedTokens;`