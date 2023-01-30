`redeemedTokens` initialised at 0 is not needed 

code: [https://github.com/rabbitholegg/quest-protocol/blob/9ef32907788dde6c42990ee5dde8f53caeaba474/contracts/Quest.sol#L22](https://github.com/rabbitholegg/quest-protocol/blob/9ef32907788dde6c42990ee5dde8f53caeaba474/contracts/Quest.sol#L22)

empty block should be made moved or be made into an interface

code: [https://github.com/rabbitholegg/quest-protocol/blob/9ef32907788dde6c42990ee5dde8f53caeaba474/contracts/Quest.sol#L150](https://github.com/rabbitholegg/quest-protocol/blob/9ef32907788dde6c42990ee5dde8f53caeaba474/contracts/Quest.sol#L150)

The ticket can be set to 0x00 address

Code: [https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L54](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L54)

Code: [https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L60](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L60)

Code: [https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L66](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L66)

Code: [https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L73](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L73)

Mitigation: `require( ticketRenderer_ != address(0))`