
[https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Quest.sol#L104](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Quest.sol#L104)

[https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Quest.sol#L69](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Quest.sol#L69)

[https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleReceipt.sol#L117](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleReceipt.sol#L117)

[https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleReceipt.sol#L128](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleReceipt.sol#L128)

For loop optimization:

[1]  An arrays length should be cached to save gas on for loops.

[2]  uint i = 0 is unnecessary as the default value is 0 anyway.

[3] Use pre-fixed increments ++1 instead of i++
