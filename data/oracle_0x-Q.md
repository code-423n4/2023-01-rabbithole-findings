# Lack of zero address validation

## Handle
oracle_0x

## Vulnerability details
### Impact
The initialize() function of several files such as QuestFactory.sol, RabbitHoleReceipt, RabbitHoleTickets lack zero address validation for the address parameters. 

Since they are used to initialize state address variables which are used in other functions of the system, the wrong address being set like the zero address can lead to interaction errors which might cause denial of service like revert.

It can be noted that the project can set the address parameters later with the set_XX methods. Also, to give the project credit, the setProtocolFeeRecipient method has a zero address validation [Ref]( https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L165)

### Proof of Concept
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L43
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L32

### Tools Used
Manual Review

### Recommended Mitigation Steps
Add a check for zero address

