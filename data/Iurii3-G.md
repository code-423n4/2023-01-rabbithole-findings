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


### [G-05] May assign the same role twice

There is no check if caller provided right bool canCreateQuest_ as an argument. 
We may accidentally grant/revoke role second time and lose gas.

[QuestFactory.sol#L142](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L142)

We can add check by updating if statement from 

    if (canCreateQuest_)
to

    if (canCreateQuest_ && hasRole(CREATE_QUEST_ROLE, account_) ) 



### [G-06]  Possible quest Id collision 

While creating quest any sting may be used as it's ID. 
Using the same ID second time will be reverted  as QuestIdUsed(). 
However, there is no direct way to retrieve if current current Quest Id used or not (it is possible to use questInfo() function, but it will retrieve (0, 0, 0) rather then bool if the current id is available.

It might be useful to create view function that checks if current quest id is available.

[QuestFactory.sol#L68](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L68)


### [G-07] <x> += <y>  COSTS MORE GAS THAN <x> = <x> + <y>  FOR STATE VARIABLES


Using the addition operator instead of plus-equals saves gas

[Quest.sol#L115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115)


### [G-08] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

Make it outside and only use it inside.

[Quest.sol#L70](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70)
[Quest.sol#L101](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L101)
[Quest.sol#L104](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104)


### [G-09] Change i++ to ++i


[Quest.sol#L70](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70)
[Quest.sol#L104](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104)
[Quest.sol#L106](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L106)
[QuestFactory.sol#L226](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L226)
[RabbitHoleReceipt.sol#L117](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117)
[RabbitHoleReceipt.sol#L121](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L121)
[RabbitHoleReceipt.sol#L128](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128)
[RabbitHoleReceipt.sol#L131](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L131)