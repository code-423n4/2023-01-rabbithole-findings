## Gas Optimizations
### [GAS-1] Don't hash both sides of an equality to compare
Instead of hashing values for both sides of an equality, you can compare the values directly. For example, instead of `keccak256(valueA) == keccak256(valueB)`, it is possible to use `valueA == valueB`. This will result in the same boolean outcome, without incurring the gas cost of keccak256 hashing.

<i>3 Instances</i>
```solidity
File: QuestFactory.sol

72:		if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {

105:    if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
```

```solidity

File: RabbitHoleReceipt.sol

119:	if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
```

### [GAS-2] Use immutable values where possible
When a value is assigned to state in a constructor and is never modified afterwards, it is more gas efficient to make it immutable.

<i>1 Instance</i>
```solidity
File: Quest.sol

21:	string public questId;
```