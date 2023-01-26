### typo
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L176

typo in remove

### type recommendations
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L84

uint unclaimedTokens should be made uint256 to avoid confusion

receiptRedemeers() returns uint256, rewardAmountInWeiOrTokenId_ is a uint256 value and redmeedTokens is uint256. So to avoid confusion make unclaimedTokens uint256

### type recommendations 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L21

change uint to uint256

### type recommendations
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L24

change uint to uint256