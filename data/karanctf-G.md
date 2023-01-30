## [G-1]
On file [QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72) on line 72 and 105 insted of calculating constants like keccak256(abi.encodePacked('erc20'))) on the fly precalculate them and store to save gas like
```solidity
if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
//0x5a28e9363bb942b639270062aa6bb295f434bcdfc42c97267bf003f272060dc9// as byte32
.
.
if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
```


