## [G-1]  Refund gas optimization

While burning token here:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L140
fields in mappings does not clearing
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L30
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L34
*Recommendation:*
you may save up to 1/5 of transaction cost with gas refund if rewriting burn function with:

```solidity
super._burn(tokenId_);
delete questIdForTokenId[tokenId_];
delete questIdForTokenId[timestampForTokenId];
```

----

## [G-2]  Redundant modifier

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81
```withdrawRemainingTokens``` from ```Quest.sol``` already has ```onlyOwner``` modifier.
*Recommendation:*
Remove ```onlyOwner``` modifier from ```ERC20Quest.sol```'s ```withdrawRemainingTokens```
function.


----

## [G-3]  Use Bitmap for claimedList

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L24

```solidity
mapping(uint256 => bool) private claimedList;
```

You're writing information about claim token in a single slot it cost 20_000 gas.
You may Bitmap (like in
UniswapV3 https://github.com/Uniswap/v3-core/blob/main/contracts/libraries/TickBitmap.sol) and save
up to 15_000 gas per token further.
*Recommendation:*
Rewrite to map like this

```solidity
mapping(uint256 => uint256) private claimedList;
``` 

Add function position:

```solidity
function position(uint256 tokenId) private pure returns (int16 wordPos, uint8 bitPos) {
    wordPos = int16(tokenId >> 8);
    bitPos = uint8(tokenId % 256);
}
```

Next add function to check is token claimed and function to claim token.
Use https://github.com/Uniswap/v3-core/blob/main/contracts/libraries/TickBitmap.sol as a sample.

----

## [G-4]  Redundant call

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L43
With current implementation ```__AccessControl_init``` has empty body and doesn't have any side
effects.
*Recommendation:*
Remove call:

```solidity
__AccessControl_init();
```

----

## [G-5]  Redundant encoding

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L105
All samples above has strings which are encodes and hashes, but result from call to call will be
the same.
*Recommendation:*
Make it constant and use futher

```solidity
bytes32 private constant ERC11555_HASH = keccak256(abi.encodePacked('erc1155'));
bytes32 private constant ERC20_HASH = keccak256(abi.encodePacked('erc20')); 
```

----

## [G-6]  Redundant field

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L222
Field ```hash_``` is redundant because it already contains in ```signature``` by signing
algorithm definition.
*Recommendation*:
Instead of comparing:

```solidity
if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
```  

You may just create hash, pass it to ```recoverSigner``` function and compare
with ```claimSignerAddress```.

----

## [G-7]  Unused inheritance

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L17
Contract ```ERC721Upgradeable``` already contains in ```ERC721EnumerableUpgradeable```
and ```ERC721URIStorageUpgradeable```.
*Recommendation:*
Remove definition of ```ERC721Upgradeable``` it saves gas during deploy and improves readability.


----

## [G-8]  Unused mapping

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L30

```solidity
mapping(uint => string) public questIdForTokenId;
```

*Recommendation:*
Remove this mapping and this map filling.

----

## [G-9]  Non optimal definition:

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L119
In line

```solidity
if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_)))
```

You're comparing ```questIdForTokenId[tokenId])``` with ```questId_``` on each cycle and
use ```keccak256``` for ```questId_```.
You can make hashing before the loop and just compare hash result in loop.
*Recommendation:*
Rewrite like:

```solidity
bytes32 hash = keccak256(bytes(questId_));

for (uint i = 0; i < msgSenderBalance; i++) {
    uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
    if (keccak256(bytes(questIdForTokenId[tokenId])) == hash) {
        tokenIdsForQuest[i] = tokenId;
        foundTokens++;
    }
}
```

----

## [G-10]  Non optimal memory use and cycle iteration

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L109-L135
In this function you iterate over whole sender's token twice. But you need two iterations: The
first one over all user's tokens, the second one over filtered tokens only.
*Recommendation:*
Write into last free position of ```tokenIdsForQuest``` instead of actual position. Rewrite like
this:

```solidity
for (uint i = 0; i < msgSenderBalance; i++) {
    uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
    if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
        tokenIdsForQuest[foundTokens] = tokenId;
        foundTokens++;
    }
}
```

Then rewrite second loop to:

```solidity
for (uint i = 0; i < foundTokens; i++) {
    filteredTokens[i] = tokenIdsForQuest[i];
}
```

--- 

## [G-11] Loads struct into memory to save gas

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L215-L229

use next pattern:
```solidity

Quest memory quest = quests[questID_];
```

Next read from this struct, *but remember that you must write into storage*
  
