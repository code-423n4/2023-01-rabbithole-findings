QA1. The NatSpec here is wrong
```
@dev See {IERC165-royaltyInfo}
```
The correction is
```
@dev See {IERC2981Upgradeable-royaltyInfo}

```

QA2. It is advisable to have a zero address check for all the setters such as: ``setMinterAddress()``, ``setQuestFactory()``, ``setReceiptRenderer``, etc. 

QA3. The owner has overly-centralized control in the QuestFactory contract, including 1) ``changeCreateQuestRole()``, 2) ``setClaimSignerAddress()``, 3) ``setClaimSignerAddress()``, 4) ``setRabbitHoleReceiptContract()``, 5) ``setRewardAllowlistAddress`` 6) ``setRewardAllowlistAddress()``, and 7)  ``changeCreateQuestRole()``
A balance-and-check should be introduced. 


