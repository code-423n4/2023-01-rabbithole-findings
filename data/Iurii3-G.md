### [G-01] Unnecessary check

If the second check would be passed (case quests[questId_].addressMinted[msg.sender] == false) the first would be always passed as well. 
May leave the second one only.

[QuestFactory.sol#L220-L221](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L220-L221)


### [G-02] Unnecessary variable assignment

isPaused is false by default and there is no way to un-pause it before calling start() function

[Quest.sol#L51](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L51)


### [G-03] Unused argument of a function 

[RabbitHoleTickets.sol#L110](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L110)


### [G-04] Mapping not used

There is a mapping between timestamp and receipt token id. 
However it is not clear how we would use it without any view function to return information from it.
Consider deleting or adding view function

[RabbitHoleReceipt.sol#L34](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L34)


### [G-05] <x> += <y>  COSTS MORE GAS THAN <x> = <x> + <y>  FOR STATE VARIABLES


Using the addition operator instead of plus-equals saves gas

[Quest.sol#L115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115)


### [G-06] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

Make it outside and only use it inside.

[Quest.sol#L70](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70)
[Quest.sol#L101](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L101)
[Quest.sol#L104](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104)


### [G-07] Change i++ to ++i


[Quest.sol#L70](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70)
[Quest.sol#L104](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104)
[Quest.sol#L106](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L106)
[QuestFactory.sol#L226](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L226)
[RabbitHoleReceipt.sol#L117](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117)
[RabbitHoleReceipt.sol#L121](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L121)
[RabbitHoleReceipt.sol#L128](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128)
[RabbitHoleReceipt.sol#L131](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L131)