https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol

Add Natspect comments to IQuest for clarity of the interface, it's events, errors and functions


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol

Add Natspect comments to IQuest for clarity of the interface, it's events, errors and functions


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

Single step ownership handover is dangerous and prone to error, rather use double step ownership where newOwner has to claim ownership while oldOwner retains ownership until new Owner claims it.




