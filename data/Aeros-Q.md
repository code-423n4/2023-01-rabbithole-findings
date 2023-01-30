### Low Severity Findings:

The contracts should use a fixed version pragma of Solidity rather than a floating pragma. All contracts in scope currently use:

    pragma solidity ^0.8.15;

The _\_burn()_ function in **[RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol)** is declared "internal" but is never used: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L137-L141

---

### Non-Critical Findings:

Inconsistent capitalization styling for variables with "RabbitHole" in the name. In most places, "rabbit" and "hole" are treated as 2 different words and so the variable name follows mixedCase styling (e.g. _rabbitHoleReceiptContract_). However, in [**QuestFactory.sol**](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol) this is not the case; "h" is never capitalized in variable names.

Likewise, these 2 variables in **[RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol)** are not in mixedCase either:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L35
