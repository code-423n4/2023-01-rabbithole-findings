# Table of contents
- [G-01] SETTING THE CONSTRUCTOR TO PAYABLE
- [G-02] USE ASSEMBLY TO CHECK FOR ADDRESS(0)
- [G-03] INSTEAD OF CALCULATING A STATEVAR WITH KECCAK256() EVERY TIME THE CONTRACT IS MADE PRE CALCULATE THEM BEFORE AND ONLY GIVE THE RESULT TO A CONSTANT



## [G-01] SETTING THE CONSTRUCTOR TO PAYABLE
Saves ~13 gas per instance

##  [G-02] USE ASSEMBLY TO CHECK FOR ADDRESS(0)
Saves 6 gas per instance:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L70
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L166
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L162


## [G-03] INSTEAD OF CALCULATING A STATEVAR WITH KECCAK256() EVERY TIME THE CONTRACT IS MADE PRE CALCULATE THEM BEFORE AND ONLY GIVE THE RESULT TO A CONSTANT

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L17


