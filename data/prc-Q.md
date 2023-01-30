## Summary

In the project code some redundant modifiers, and some usage of literal numbers instead of constants were found.

## Findings

### Low Severity

- **Redundant Modifier** - redundant `onlyOwner` modifier since super function is explicitly called:

|`quest-protocol/contracts/Erc1155Quest.sol`|
|---|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54|

|`quest-protocol/contracts/Erc20Quest.sol`|
|---|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81|


### Non-critical

- **Use of literals instead of constants** - the use of constants instead of literal numbers is recommended, as they improve code readability:

|`quest-protocol/contracts/QuestFactory.sol`|
|---|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L48|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L187|

|`quest-protocol/contracts/RabbitHoleReceipt.sol`|
|---|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L184|

|`quest-protocol/contracts/RabbitHoleTickets.sol`|
|---|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L113|

- **Use of _ in number literals** - the use of `_` in numbers is a means of improving readability, and as the literals in the project are in percent basis points, the recommended placement would be like `100_00`, instead of the `10_000` that is used:

|`quest-protocol/contracts/QuestFactory.sol`|
|---|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L48|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L187|

|`quest-protocol/contracts/RabbitHoleReceipt.sol`|
|---|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L184|

|`quest-protocol/contracts/RabbitHoleTickets.sol`|
|---|
|https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L113|

(This list is not exhaustive, maybe other places could be found)
