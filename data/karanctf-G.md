## [G-1]
On file [QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72) on line 72 and 105 instead of calculating constants like keccak256(abi.encodePacked('erc20'))) on the fly precalculate them and store to save gas like
```solidity
if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
//0x5a28e9363bb942b639270062aa6bb295f434bcdfc42c97267bf003f272060dc9// as byte32
.
.
if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
```


## [G-2]
On [claim()](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96) function after the tokens are checked which are claimed and which are not then memory array of `token` is passed in a function` _setClaimed(tokens);`  to mark them claimed one by one. 
```solidity
function _setClaimed(uint256[] memory tokenIds_) private {

	for (uint i = 0; i < tokenIds_.length; i++) {

		claimedList[tokenIds_[i]] = true;

	}

}
```

Insted of this it can be done on the time of checking if they are claimed or not on line [106](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L106) 
It takes only memory array of tokens.
```solidity
function claim() public virtual onlyQuestActive {

	if (isPaused) revert QuestPaused();

	uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);

	if (tokens.length == 0) revert NoTokensToClaim();

	uint256 redeemableTokenCount = 0;

	for (uint i = 0; i < tokens.length; i++) {

		if (!isClaimed(tokens[i])) {

			redeemableTokenCount++;// also use preincrement here
++			claimedList[tokens[i]] = true;// set them here directly

		}

	}

if (redeemableTokenCount == 0) revert AlreadyClaimed();

uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);

_setClaimed(tokens);

_transferRewards(totalRedeemableRewards);

redeemedTokens += redeemableTokenCount;

emit Claimed(msg.sender, totalRedeemableRewards);

}
```
