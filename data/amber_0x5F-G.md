## quests[questId_].numberMinted++ should be unchecked {quests[questId_].numberMinted++}.

At : https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L226

the numberMinted value is checked at :

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L220

the totalParticipants is already set at quest creation and there is no entrance to change it, so that it is impossible to overflow for the value of numberMinted. 

Use unchecked will save gas.

## Variable `questId` should be immutable in Quest.sol.
At : https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L21

Variable `questId` does not change once set in the constructor, immutable will save gas when read the variable at : 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L99

## Variable `redeemedTokens` may not be assigned 0 in the constructor in Quest.sol.
At : https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L45

`redeemedTokens` was originally 0 when the Quest was created. Delete the code is fine ans saves gas.