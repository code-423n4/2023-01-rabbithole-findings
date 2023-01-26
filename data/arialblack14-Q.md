# QA REPORT

---

### Summary of low risk issues


| Number     | Issue details                              | Instances |
| ------------ | -------------------------------------------- | :---------: |
| [L-1](#L1) | Use of`block.timestamp`                    |     5     |
| [L-2](#L5) | Lock pragmas to specific compiler version. |    10    |

*Total: 2 issues.*

### Summary of non-critical issues


| Number       | Issue details                                                                         | Instances |
| -------------- | --------------------------------------------------------------------------------------- | :---------: |
| [NC-1](#NC1) | Constants should be defined rather than using magic numbers                           |     6     |
| [NC-2](#NC2) | Use scientific notation (e.g.`1e18`) rather than large multiples of 10 (e.g. 1000000) |     5     |
| [NC-3](#NC3) | Lines are too long.                                                                   |     3     |

*Total: 3 issues.*

---

### Low Risk Issues

### <a id=L1>[L-1]</a> Use of `block.timestamp`

##### Description

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the *[Entropy Illusion](https://hacken.io/discover/most-common-smart-contract-vulnerabilities/#Entropy_Illusion)* for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

##### Recommendation

Use oracle instead of `block.timestamp`

##### *Instances (5):*

File: [2023-01-rabbithole/quest-protocol/contracts/Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L35 )

```solidity
35: if (endTime_ <= block.timestamp) revert EndTimeInPast();
36: if (startTime_ <= block.timestamp) revert StartTimeInPast();
77: if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
90: if (block.timestamp < startTime) revert ClaimWindowNotStarted();
```

File: [2023-01-rabbithole/quest-protocol/contracts/RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L102 )

```solidity
102: timestampForTokenId[newTokenID] = block.timestamp;
```

### <a id=L5>[L-2]</a> Lock pragmas to specific compiler version.

##### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

##### Recommendation

Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version. [solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

##### *Instances (10):*

File: [2023-01-rabbithole/quest-protocol/contracts/Erc1155Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L2 )

```solidity
2: pragma solidity ^0.8.15;
```

File: [2023-01-rabbithole/quest-protocol/contracts/Erc20Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L2 )

```solidity
2: pragma solidity ^0.8.15;
```

File: [2023-01-rabbithole/quest-protocol/contracts/Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L2 )

```solidity
2: pragma solidity ^0.8.15;
```

File: [2023-01-rabbithole/quest-protocol/contracts/QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L2 )

```solidity
2: pragma solidity ^0.8.15;
```

File: [2023-01-rabbithole/quest-protocol/contracts/RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L2 )

```solidity
2: pragma solidity ^0.8.15;
```

File: [2023-01-rabbithole/quest-protocol/contracts/RabbitHoleTickets.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L2 )

```solidity
2: pragma solidity ^0.8.15;
```

File: [2023-01-rabbithole/quest-protocol/contracts/ReceiptRenderer.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L2 )

```solidity
2: pragma solidity ^0.8.15;
```

File: [2023-01-rabbithole/quest-protocol/contracts/TicketRenderer.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L2 )

```solidity
2: pragma solidity ^0.8.15;
```

File: [2023-01-rabbithole/quest-protocol/contracts/interfaces/IQuest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L2 )

```solidity
2: pragma solidity ^0.8.15;
```

File: [2023-01-rabbithole/quest-protocol/contracts/interfaces/IQuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L2 )

```solidity
2: pragma solidity ^0.8.15;
```

### Non-critical Issues

### <a id=NC1>[NC-1]</a> Constants should be defined rather than using magic numbers

##### Description

Even [`assembly`](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

##### Recommendation

You should declare the variable instead of magic number.

##### *Instances (6):*

File: [2023-01-rabbithole/quest-protocol/contracts/Erc20Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L53 )

```solidity
53: return (maxTotalRewards() * questFee) / 10_000;
97: return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
```

File: [2023-01-rabbithole/quest-protocol/contracts/QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L187 )

```solidity
187: if (questFee_ > 10_000) revert QuestFeeTooHigh();
```

File: [2023-01-rabbithole/quest-protocol/contracts/RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L129 )

```solidity
129: if (tokenIdsForQuest[i] > 0) {
184: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

File: [2023-01-rabbithole/quest-protocol/contracts/RabbitHoleTickets.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L113 )

```solidity
113: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

### <a id=NC2>[NC-2]</a> Use scientific notation (e.g.`1e18`) rather than large multiples of 10 (e.g. 1000000)

##### Description

Use scientific notation (e.g.`1e18`) rather than large multiples of 10, that could lead to errors.

##### Recommendation

Use scientific notation instead of large multiples of 10.

##### *Instances (5):*

File: [2023-01-rabbithole/quest-protocol/contracts/Erc20Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L50 )

```solidity
53: return (maxTotalRewards() * questFee) / 10_000;
97: return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
```

File: [2023-01-rabbithole/quest-protocol/contracts/QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L187 )

```solidity
187: if (questFee_ > 10_000) revert QuestFeeTooHigh();
```

File: [2023-01-rabbithole/quest-protocol/contracts/RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L184 )

```solidity
184: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

File: [2023-01-rabbithole/quest-protocol/contracts/RabbitHoleTickets.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L113 )

```solidity
113: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

### <a id=NC3>[NC-3]</a> Lines are too long.

##### Description

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length

##### Recommendation

Reduce number of characters per line to improve readability.

##### *Instances (3):*

File: [2023-01-rabbithole/quest-protocol/contracts/Erc20Quest.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L56 )

```solidity
56: /// @notice Starts the quest by marking it ready to start at the contract level. Marking a quest ready to start does not mean that it is live. It also requires that the start time has passed
57: /// @dev Requires that the balance of the rewards in the contract is greater than or equal to the maximum amount of rewards that can be claimed by all users and the protocol
```

File: [2023-01-rabbithole/quest-protocol/contracts/interfaces/IQuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L16 )

```solidity
16: event QuestCreated(address indexed creator, address indexed contractAddress, string indexed questId, string contractType, address rewardTokenAddress, uint256 endTime, uint256 startTime, uint256 totalParticipants, uint256 rewardAmountOrTokenId);
```
