| Finding |Details|Context|
|:--:|:-------|:--:|
|[L-01]| Missing sanity check for royalty fee  | 2 |
|[N-01]| Useful information in events should be indexed | 1 |
|[N-02]| Don't compare a boolean with `true` | 2 |

## [L-01] Missing sanity check for royalty fee.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66-L69

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90-L93

According to the function `royaltyInfo`, the royalty fee is measured in BPS. However there is no sanity check for the amount of fee, meaning the owner can set it arbitrarily high. This is recognized as an issue given there is such a check on `QuestFactory`.

## [N-01] Useful information in events should be indexed

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L8

The following event can use an `indexed` field into `address account`. Although indexed fields will incur slightly more gas (375 gas per field per event), it will make it easy to query useful info specific to the field (e.g. querying how many claims has this user done, total amounts, and such)

```solidity 
event Claimed(address account, uint256 amount);
```

## [N-02] Don't compare a boolean with `true`

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L113

A boolean expression in an if-condition is straightforwardly evaluated. There is no need to check `if (condition == true)`. 

Removing the check will also save some gas.