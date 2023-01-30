## [GAS-01] UNCHECKING ARITHMETICS OPERATIONS THAT CAN’T UNDERFLOW/OVERFLOW
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.
[Reference](https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic)
```
RabbitHoleReceipt.sol:
117: for (uint i = 0; i < msgSenderBalance; i++) {
128: for (uint i = 0; i < msgSenderBalance; i++) {

Quest.sol: 
70: for (uint i = 0; i < tokenIds_.length; i++) {
104: for (uint i = 0; i < tokens.length; i++) {
```
***

## [GAS-02] USE NAMED RETURNS FOR LOCAL VARIABLES WHERE IT IS POSSIBLE
```
RabbitHoleReceipt.sol:
109: function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory) {
	
Quest.sol:
140: function getRewardAmount() public view returns (uint256) {
145: function getRewardToken() public view returns (address) {
```
***

## [GAS-03] `x = x - y` costs less gas than `x -= y`, same for addition
```
Quest.sol
115: redeemedTokens += redeemableTokenCount;
```
You can replace all `-=` and `+=` occurrences to save gas
***