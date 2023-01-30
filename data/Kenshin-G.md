# Boolean expressions cost less gas than boolean literals
## Permalinks
* https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L136
* https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221
* https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73

## Description
`if (x == true)` costs more gas than `if (x)`. There is no need to compare boolean as literals.

## Remediation
Use boolean expression instead. `if (x) { //x == true }` or `if (!x) { //x == false }`
