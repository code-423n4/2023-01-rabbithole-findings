## Setting the `constructor` to `payable` can save gas 

Setting constructor to payable will save ~13 gas per instance.

### Proof of Concept

```Solidity

35: constructor() initializer {}

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L35

```Solidity

39: constructor() {
		_disableInitializers();
	}

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L39

```Solidity

26:	constructor(
		address rewardTokenAddress_,
		uint256 endTime_,
		uint256 startTime_,
		uint256 totalParticipants_,
		uint256 rewardAmountInWeiOrTokenId_,
		string memory questId_,
		address receiptContractAddress_
	)

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L26

```Solidity

28:	constructor() {
	
		_disableInitializers();
	
	}

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L28

```Solidity

17:	constructor(
		address rewardTokenAddress_,
		uint256 endTime_,
		uint256 startTime_,
		uint256 totalParticipants_,
		uint256 rewardAmountInWeiOrTokenId_,
		string memory questId_,
		address receiptContractAddress_,
		uint256 questFee_,
		address protocolFeeRecipient_
	)

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L17

```Solidity

13:	constructor(
		address rewardTokenAddress_,
		uint256 endTime_,
		uint256 startTime_,
		uint256 totalParticipants_,
		uint256 rewardAmountInWeiOrTokenId_,
		string memory questId_,
		address receiptContractAddress_
)

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L13```

### Recommended Mitigation Steps

Add the `payable` keyword.

## Save gas with `unchecked` keyword

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.

### Proof of Concept

```Solidity

117: for (uint i = 0; i < msgSenderBalance; i++) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117


```Solidity

128: for (uint i = 0; i < msgSenderBalance; i++) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128


```Solidity

70: for (uint i = 0; i < tokenIds_.length; i++) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70


```Solidity

104: for (uint i = 0; i < tokens.length; i++) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104

### Recommended Mitigation Steps

Use `uncheked{++i}`

## `internal` function only called once can be inlined to save gas

### Proof Of Concept

```Solidity

152: function grantDefaultAdminAndCreateQuestRole(address account_) internal {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152

## Save gas with saving storage variable to memory

The instances below point to the second access of a storage variable within a function.
`quests[questId_].numberMinted`, `quests[questId_].addressMinted[msg.sender]` in `function mintReceipt` 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219

### Recommended Mitigation Steps

Add this lines before first use and replace the corresponding variables next:
`uint256 numberMinted = quests[questId_].numberMinted;`
`address addressMinted = quests[questId_].addressMinted[msg.sender]`

Add these lines after the last use:
`quests[questId_].numberMinted = numberMinted;`
`quests[questId_].addressMinted[msg.sender] = addressMinted;`
