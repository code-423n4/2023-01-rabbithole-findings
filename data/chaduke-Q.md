QA1. The NatSpec here is wrong
```
@dev See {IERC165-royaltyInfo}
```
The correction is
```
@dev See {IERC2981Upgradeable-royaltyInfo}

```

QA2. It is advisable to have a zero address check for all the setters such as: ``setMinterAddress()``, ``setQuestFactory()``, ``setReceiptRenderer``, etc. 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L54-L62


QA3. The owner has overly-centralized control in the QuestFactory contract, including 1) ``changeCreateQuestRole()``, 2) ``setClaimSignerAddress()``, 3) ``setClaimSignerAddress()``, 4) ``setRabbitHoleReceiptContract()``, 5) ``setRewardAllowlistAddress`` 6) ``setRewardAllowlistAddress()``, and 7)  ``changeCreateQuestRole()``

Mitigation: A balance-and-check should be introduced so that each manager is responsible only for a couple of tasks. 

QA4. 
The following modifier name is not consistent with its implementation logic. Better change it to ``onlyAfterQuestEnd()``.  

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76-L79

```
 modifier onlyAdminWithdrawAfterEnd() {
        if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
        _;
    }
```


QA5: There is a contradictory design  in terms of the number of rabbitHoleReceipts a user can win. 


Analysis: In one place, it allows MANY: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L99
```
 uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);
```
which retrieves a whole list of wined receipt tokens. 

In another place, it is restricted to ONE:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221
```
 if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```
Each address can mint at most once. 

Mitigation: In general, we should allow a user to win multiple rabbitHoleReceipts. Restrictions, if any, should be enforced on a per quest basis, not across-the-board. To do this, eliminate the following check in ``mintReceipt()``.

```
if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();

```

QA6. Signature replay attack is possible for ``mintReceipt()`` such that a malicious user can mint receipts for free. 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219

Analysis: This is because the signature only signs for ``(winnerAddress, questId)``, so the signature can be reused for another contract and another blockchain to mint free receipts. 

Mitigation: To prevent signature replay attack, include nonce, contract address and blockchain ID in the hash as well. 

QA7. ``royaltyFee`` should never be greater than 100% of the salePrice (represented as 10_000). 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66-L69

The mitigation is to add such a check:
```
 function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        if(royaltyFee > 10_000) revert royaltyFeeTooLarge();   // audit: add this check 

        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
```

