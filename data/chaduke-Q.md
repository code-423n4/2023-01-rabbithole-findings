QA1. The NatSpec here is wrong, this is not ERC165, it is about ERC2981
```
@dev See {IERC165-royaltyInfo}
```
The correction is
```
@dev See {IERC2981Upgradeable-royaltyInfo}

```

QA2. When admins/owner set important address parameters, it is important to do a zero address check and also 
``X != address(this)`` check to ensure not losing funding due to mistakes. 

1) https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71-L73
2) https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L77-L79
3) https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L83-L86
4) https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L65-L67


QA3. The ``withdrawFee()`` has the risk of sending protocol fee to a dead address: 

https://github.com/rabbitholegg/quest-protocol/blob/9ef32907788dde6c42990ee5dde8f53caeaba474/contracts/Erc20Quest.sol#L102-L104

The losing of fund can happen in the following way:
1) A dead address is set as the protocol recipient by the owner of ``QuestFactory`` by mistake or by a compromised owner on purpose. 

2)  The malicious user calls ``withdrawFee()`` and then protocol fee is sent to the dead address. 

3) Nobody can retrieve the protocol fee from the dead address. 

Mitigation: add a modifier so that only the protocol recipient can call  ``withdrawFee()``.  In this way, no fund will be sent to a dead address. 

QA4. 
The following modifier name is not consistent with its implementation logic. Better change it to ``onlyAfterQuestEnd()``.  

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76-L79

```
 modifier onlyAdminWithdrawAfterEnd() {
        if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
        _;
    }
```


QA5. Signature replay attack is possible for ``mintReceipt()`` such that a malicious user can mint receipts for free. 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219

Analysis: This is because the signature only signs for ``(winnerAddress, questId)``, so the signature can be reused for another contract and another blockchain to mint free receipts. 

For example, if a quest was very successful on Ethererum and then it is  deployed on a new blockchain Polygon,
one can reuse the signature signed on the Ethereum to get free receipt on Polygon by calling ``mintReceipt`` with the same signature. 

Mitigation: To prevent signature replay attack, include nonce, contract address and blockchain ID in the hash as well. 

QA6. ``royaltyFee`` should never be greater than 100% of the salePrice (represented as 10_000). Otherwise, the seller gets nothing from a sale or causing the selling to fail.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66-L69



The mitigation is to add such a check:
```
 function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        if(royaltyFee > 10_000) revert royaltyFeeTooLarge();   // audit: add this check 

        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
```

QA7. If a sponsor sends the wrong ERC20/ERC1155 to the quest contract (e.g. due to miscommunication), for example,  instead of sending USDC, the sponsor sends USDT as reward tokens, then they will be locked in the contract forever since there is no way to withdraw arbitrary ERC20/ERC1155 tokens. 

Mitigation: introduce generic ``withdrawERC20(address tokenAddress)`` and ``withdraw1155(address tokenAddress)``so that the host can withdraw arbitrary ERC20/ERC1155 tokens. 

QA8. The mint() function of the RabbitHoleReceipt violates the separation of duty rule. The minter of one quest can mint tokens for another quest.
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L98-L104
For example, Alice is the minter for quest A, so she has the role of minter, then she can also call mint(to, B) to mint receipts for quest B, as a result, she can claim rewards from quest B that has nothing to do with quest A.

Mitigation: each quest should has its own minter. A minter of quest A should be not allowed to mint receipts for quest B. 

QA9. ``QuestIdCount`` is always the total number of Quests + 1. This is confusing. 

This is because it is initialized to 1 in the constructor: 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L49
```
 questIdCount = 1;
```
and it increases by 1 each time a new quest is launched:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L101
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L132
```
 ++questIdCount;
```
Mitigation: don't bother to initialize it to 1 in the constructor, so it will be zero initially. 

QA10.  It is important to declare a uint _gap[50] state variable for the following upgradable implementation contracts so that when they are upgraded with the introduction of new state variables, other inheriting contracts will not be disturbed. 

1)  QuestFactory
2) RabbitHoleReceipt 
3) RabbitHoleTickets
