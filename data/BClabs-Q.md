# 1 Fee should not be settable to over 10_000 bps in ERC2981

When setting royaltyFee, there is no limitation for admin to set it to any amount even greater than 10_000 bps, which would result in marketplaces not being able to send royalties.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90-L93

# 2 Airdrops stuck in a contract

Since the quest is holding ERC20 or ERC1155 tokens, what could happen is that the project would send out the tokens, which would get stuck in the contract, since there is no function to sweep them.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L12

# 3 In the case of ERC1155, the reward can only be set to 1

QuestOwner can only set the reward for ERC1155 quest to be 1 unit. 

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L48-L50

# 4 ECDSA imported but never used

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L7

# 5 Change of claimSignerAddress can result in the user not being able to mint their Receipt

1. Let's assume Alice finishes the quest and receives the receipt. 
2. At that moment signerAddress gets changed.
3. Alice is therefore unable to use the signature since the signer changed.

To mitigate this, do not allow signer changes or use it with caution.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159-L161

# 6 Starting questIdCount with 1 is misleading

The counter should start with 1, otherwise, it is misleading since the user can assume there has been 1 quest deployed at the beginning.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L49

# 7 Receipt's SVG is broken 

Texts in SVG are overlapping since y is set to 40% in both lines in generateSVG
Should be: 
```
x="50%" y="40%"
x="50%" y="50%"
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L105-L108

# 8 Length of quest id should be limited to a certain length, otherwise text in SVG overflows.

If quest id's string is too long, then the NFT will seem broken. For example "MMMMMMMMMMMMMMMMMMMMM" will overflow. It is difficult to calculate text size since every char has its own size, so monospaced font should be used and limit the text length, so it fits the rect element.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L106
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L22

# 9 Max tokenId_ can have 32 chars in order to fit the SVG, then it overflows. A condition should be used so that this number is not crossed.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L42
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L109

# 10 getRewardAmount returns the wrong number in the case of ERC1155, it returns the token id instead of the amount.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L140-L142

# 11 Uri should only be returned for existent (minted) NFTs. 

This can cause aggregators to show false data since they could assume that an NFT exists even if it hasn't been minted yet.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L102-L105

# 12. name onlyAdminWithdrawAfterEnd is deceiving since anyone can call the function with this modifier not only admin.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76

