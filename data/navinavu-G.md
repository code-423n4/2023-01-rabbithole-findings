## The use of a `string` `QuestId` type instead of an unsigned integer type leads to an excessive use of `keccak256` in loop.
In the `RabbitHoleReceipt.sol` contrat, the `getOwnedTokenIdsOfQuest()` function has to make sure the `questId_` parameter matches 
the `questIdForTokenId[tokenId]` value by using the keccak256 hashing function. 
As the keccak256 function is called inside the for-loop, it is using a basis of 30 units of gas + 6 units of gas per word on each iteration
Consider using unsigned integer types for Quests IDs as a comparison on unsigned integers would cost only 3 gas units.

### Context
- RabbitHoleReceipt.sol : getOwnedTokenIdsOfQuest() : [l:119](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L119)
- Quest.sol : StateVar : [l:21](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L21)

---

## Use Quest structure storage pointer instead of loading the full Quest structure into memory
Avoiding to load values in memory and using storage pointers may in some scenarios cost less gas.

### Context
- QuestFactory.sol : createQuest() : [l:61](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61)
	[See diff](https://www.diffchecker.com/lmcAvOgF/)
- QuestFactory.sol : mintReceipt() : [l:219](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219)
	[See diff](https://www.diffchecker.com/dmRiTJlZ/)

### Gas reduction
(Based on project's tests gas reporter)

`QuestFactory.sol:createQuest()` | Calls: 65
| minA        | minB        | maxA        | maxB        | avgA        | avgB        | avgΔ        |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| 1365853     | 1365137     | 1508358     | 1507642     | 1503759     | 1503043     | -716        |


`QuestFactory.sol:mintReceipt()` | calls: 4 
| minA        | minB        | maxA        | maxB        | avgA        | avgB        | avgΔ        |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| 1365853     | 1365137     | 1508358     | 1507642     | 288053      | 285979      | -2074       |

--- 

## Returning an array with an over-length is cheaper than creating a new array and using a for loop
The `RabbitHoleReceipt:getOwnedTokenIdsOfQuest()` is declared as view but used by state-changing function `Quest.sol:claim()` and can be simplified by removing the second loop to save gas.
It is also possible to cut out some more gas by unchecking the `i` index variable of the loop as `i` and `msgSenderBalance` are both `uint256` and can not overflow each other.
Additionally, the parameter `questId_` localisation can be changed to `calldata` as it is not altered through the function and does not need to be passed to a memory parameter of another function or returned.

### Context
- RabbitHoleReceipt.sol : getOwnedTokenIdsOfQuest() : [l:109](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L109)
[See diff](https://www.diffchecker.com/CmQbAJcf/)


### Gas reduction
(Based on project's tests gas reporter)

`Quest.sol:claim()` | Calls: 11
| minA        | minB        | maxA        | maxB        | avgA        | avgB        | avgΔ        |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| 126644      | 125744      | 155502v     | 154181      | 129639      | 128701      | -938        |

