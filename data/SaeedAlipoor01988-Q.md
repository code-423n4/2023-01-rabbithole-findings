The upgradeable contract is missing a gap storage variable to allow for new storage variables in later versions

In order to be able to add new variables to the upgradeable contract without causing storage collisions, a storage gap should be added to the upgradeable contract. If no storage gap is added, when the upgradable abstract contract introduces new variables, it may override the variables in the inheriting contract.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L16
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L15

Consider adding a storage gap at the end of the upgradeable abstract contract