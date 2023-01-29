### [NC-01] Creator of the quest may pause it forever 

Creator of the quest has a right to stop (pause forever) quest. In case there is an unclaimed reward at that time, participant won't be able to claim it.
While quest creator role is limited it should not be a big problem. However, in case of opening it for broader public, current approach should be reconsidered. 

[Quest.sol#L57](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57)
[Quest.sol#L97](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L97)


### [NC-02] minterAddress must be equal to QuestFactory address 

Under the RabbitHoleReceipt.sol contract there are two separate state variables - minterAddress and QuestFactoryContract assigned by relevant function. 

[RabbitHoleReceipt.sol#L77](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L77)
[RabbitHoleReceipt.sol#L83](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L83)

However, for proper functioning of the mintReceipt function minterAddress must be set to QuestFactoryContract address. 

[QuestFactory.sol#L219](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219)

To avoid confusion we may combine setQuestFactory and setMinterAddress in one function that uses QuestFactory address as an argument


### [NC-03] Missing contract initialization
RabbitHoleReceipt.sol does not initialize ERC721EnumerableUpgradeable 
RabbitHoleTickets.sol does not initialize IERC2981Upgradeable


### [NC-04] Make code more readable and self-explanatory

super.withdrawRemainingTokens(to_) may be changed to modifier onlyAdminWithdrawAfterEnd to be clearer 

[Erc20Quest.sol#L82](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L82) 
[Erc1155Quest.sol#L55](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L55)

modifier onlyAdminWithdrawAfterEnd() actually does not check if admin calls it, it should be renamed to onlyAfterEnd()
[Quest.sol#L76](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76)

wrong variable name, should be claimingAddress_Balance instead of msgSenderBalance
[RabbitHoleReceipt.sol#L113](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L113)
