QA1. The NatSpec here is wrong
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


QA3. The owner has overly-centralized control in the QuestFactory contract, including 1) ``changeCreateQuestRole()``, 2) ``setClaimSignerAddress()``, 3) ``setClaimSignerAddress()``, 4) ``setRabbitHoleReceiptContract()``, 5) ``setRewardAllowlistAddress`` 6) ``setRewardAllowlistAddress()``, and 7)  ``changeCreateQuestRole()``

A compromised/malicious owner can lead to:
1) Set an accomplice's address as the ``setClaimSignerAddress``, so that the accomplice can sign many receipts for free. 

2) In charge of all the ERC1155Quests, since the owner is the owner of all ERC1155Quests and steal reward tokens from any ERC1155Quest by calling ``withdrawRemainingTokens(to)``.

3) Enable the introduction of malicious ERC20/ERC1155  reward tokens by calling ``setRewardAllowlistAddress()``


Mitigation: A balance-and-check should be introduced: restrict the power of the owner, assign some privileges to other roles/admins. 

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

Mitigation: To prevent signature replay attack, include nonce, contract address and blockchain ID in the hash as well. 

QA6. ``royaltyFee`` should never be greater than 100% of the salePrice (represented as 10_000). 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66-L69

The mitigation is to add such a check:
```
 function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        if(royaltyFee > 10_000) revert royaltyFeeTooLarge();   // audit: add this check 

        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
```

QA7. If a sponsor sends the wrong ERC20/ERC1155 to the quest contract (e.g. due to miscommunication), for example,  instead of sending USDC, the sponsor sends USDT, then they will be locked in the contract forever since there is no way to withdraw arbitrary ERC20/ERC1155 tokens. 

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
