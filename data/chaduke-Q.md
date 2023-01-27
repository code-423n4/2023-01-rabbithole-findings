QA1. The NatSpec here is wrong
```
@dev See {IERC165-royaltyInfo}
```
The correction is
```
@dev See {IERC2981Upgradeable-royaltyInfo}

```

QA2. It is advisable to have a zero address check for all the setters such as: ``setMinterAddress()``, ``setQuestFactory()``, ``setReceiptRenderer``, etc. 
