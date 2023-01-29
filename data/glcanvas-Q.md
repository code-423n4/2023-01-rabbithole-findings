## [N-1] Make 100% basis points as constant

Instead of 10_000 create constant anf use it.

----

## [N-2]  Add ability to claim for another address

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96
claims to msg.sender, but address might be compromised, and therefore you might be want to send
funds for another one address.

----

## [N-3] ERC1155 doesn't support ERC20

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L11-L64
by EIP https://eips.ethereum.org/EIPS/eip-1155 ERC1155 supports ERC721 and ERC20, but current
implementation support only ERC721.
Think about rewriting code to single Quest with ERC1155.

----

## [N-4] Make ```Quest``` abstract.

mark ```_transferRewards```, ```_calculateRewards``` as abstract.

----

## [N-5] Use ```_disableInitializers``` in ```QuestFactory```

Use ```_disableInitializers``` instead of ```initializer``` modifier in constructor.

----

## [N-6] Rename ```questId``` to ```questName```

Current naming misleading, under Id implied uint256 or some integer field not string.

----

## [N-7] Make ```grantDefaultAdminAndCreateQuestRole``` private

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152

----

## [N-8] Wrong comment

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L175-L177
Royalty from ERC2981 specification.

----

## [N-9] Use camelCase for variables

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L35
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L36
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L25

## [N-10] Use ```toHexString``` for address instead of for uint256

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L60
Rewrite to:
```solidity
generateAttribute('Reward Address', Strings.toHexString(rewardAddress_))
```


## [N-11] ```toHexString``` convert a checksummed address into a non-checksummed

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L60

You may use EIP-55: https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md#specification

----

## [N-12] Typo `remave` to `remove`

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L176

----

## [L-1] Bad ```QuestFactory``` architecture

Please, create two different functions like: ```createErc20Quest``` and ```createErc1155Quest```, it
improves readability, it removes comparing with ```contractType_```. Because further when you add
more types ```createQuest``` will grow to a huge size with a huge copy paste and in which it will be
easy to make a mistake.

----

## [L-2] Bad naming ```rewardAmountOrTokenId_```

It's hard to understand, better split to two variables ```rewardsInWei```, ```rewardTokenId```. Then
create in ```Quest``` mark ```getRewardAmount()``` as abstract. Then for ```Erc1155Quest``` create
variable ```rewardTokenId```. Then for ```Erc20Quest``` create ```rewardsInWei``` variable and use
them instead of misleading ```rewardAmountInWeiOrTokenId```.

----

## [L-3] Not enough Logs

Please add events for all functions which change state. For example: ```setRabbitHoleReceiptContract```, ```setProtocolFeeRecipient``` and so on.  

----

## [L-4] Add possibility to iterates over created quests in ```QuestFactory``` 

----

