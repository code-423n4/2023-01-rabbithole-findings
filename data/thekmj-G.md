
| Number |Details|Context|Approx. gas saved|
|:--:|:-------|:--:|:--:|
|[G-01.1-3]| Tightly pack storage for better gas saving  | 3 | 57300 total |
| | `QuestFactory` contract storage |  | 4000 |
| | `struct Quest` |  | 34200 |
| | `Quest` contract storage |  | 19100 |
|[G-02]| Don't initialize storage with default value | 1 | 2900 |
|[G-03]| Redundant condition check | 1 | 25 |
|[G-04]| Don't compare a boolean with `true` | 2 | 6 |
| **Total**| | | **60231 gas saved** |


## [G-01] Tightly pack storage for better gas saving

This finding is divided into several findings due to distinction, but the main technique of storage packing is applied in all

### [G-01.1] `QuestFactory` contract storage

Instance: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L26-L32

The current storage layout is as follow:

```solidity
address public claimSignerAddress; // one slot
address public protocolFeeRecipient; // one slot
mapping(string => Quest) public quests; // one slot
RabbitHoleReceipt public rabbitholeReceiptContract; // one slot
mapping(address => bool) public rewardAllowlist; // one slot
uint public questFee; // one slot
uint public questIdCount; // one slot
```

We can notice that `questFee` and `questIdCount` cannot be too large (with `questFee` not exceeding 10000, and `questIdCount` incremented one by one). We can use smaller value types and better pack storage as follow:

```solidity
address public claimSignerAddress;
// one slot
address public protocolFeeRecipient; 
uint48 public questFee;
uint48 public questIdCount;
// end of slot
mapping(string => Quest) public quests;
RabbitHoleReceipt public rabbitholeReceiptContract;
mapping(address => bool) public rewardAllowlist;
```

Since all of these storage variables are accessed every time a quest is created, this replaces two instances of Gwarmaccess (2100 gas) with Gcoldsload (100 gas), saving a total of **4000 gas** per quest created.

This will also save some deployment gas, due to having to set up fewer storage slots than usual.

### [G-01.2] `struct Quest`

Instance: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L19-L24

The current layout for `Quest` struct is as follow:

```solidity
struct Quest {
    mapping(address => bool) addressMinted; // 1 slot
    address questAddress; // 2 slot
    uint totalParticipants; // 3 slot
    uint numberMinted; // 4 slot
}
```

Note that this is used in storage as the `quests` mapping, so this is relevant.

There is room for optimization in `totalParticipants` and `numberMinted`: 
- `totalParticipants`: Is the number of participants.
- `numberMinted`: Is incremental.

We can thus reason that the values will be reasonably small. Thus the following layout is recommended:

```solidity
struct Quest {
    mapping(address => bool) addressMinted; // 1 slot
    address questAddress; // 2 slot
    uint48 totalParticipants; // 2 slot
    uint48 numberMinted; // 2 slot
}
```

The change brings the following optimizations:
- For each quest created, converts one Gsset (20000 gas) into one Gsreset (2900 gas) by `totalParticipants` not having to write to an empty storage. Equates to **17100 gas** saved per quest.
- For the first receipt minted per quest, `numberMinted` doesn't have to write to a zero storage slot. Thus another **17100 gas** is saved in the same manner, per quest.

The total gas saving is **34200** per quest created.

### [G-01.3] `Quest` contract storage

Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L19-L22

The storage layout is as follow

```solidity
bool public hasStarted; // slot 1
bool public isPaused; // slot 1
string public questId; // slot 2
uint256 public redeemedTokens; // slot 3
```

The `redeemedTokens` is a count of the number of tokens redeemed by the user, which cannot be too large by definition. Thus we can tightly pack the storage as follow:

```solidity
bool public hasStarted; // slot 1
bool public isPaused; // slot 1
uint240 public redeemedTokens; // slot 1
string public questId; // slot 2
```

The effect are as follow:
- Since both `hasStarted` and `redeemedTokens` is accessed whenever `claim()` is (successfully) called, converts one Gcoldsload (2100 gas) into Gwarmaccess (100 gas) for **2000 gas** in total.
- Since `hasStarted` has to be already `True` for the function to not revert, a write to `redeemedTokens` will not use a Gsset (20000 gas), but rather use a Gsreset (2900 gas), saving **17100 gas** for the first claim made by the user per quest.

This saves a total of **19100 gas**.

## [G-02] Don't initialize storage with default value

Affected instances:
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L45

```solidity
redeemedTokens = 0; // @audit this is redundant
```

The given storage variable is guaranteed by logic to be default. Setting it with the same value will incur a redundant Gsreset (**2900 gas**).

This finding is similar to **[GAS-4]** of C4udit, but was not found by the automated tool.

## [G-03] Redundant condition check

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L35-L37

Note the condition check at the `Quest` constructor:

```solidity
if (endTime_ <= block.timestamp) revert EndTimeInPast();
if (startTime_ <= block.timestamp) revert StartTimeInPast();
if (endTime_ <= startTime_) revert EndTimeLessThanOrEqualToStartTime();
```

The first condition is redundant. That is, if the latter two conditions are true, then the first condition is always true. Thus we can remove the first check entirely.

Saves **25 gas** when run on hardhat gas report.

## [G-04] Don't compare a boolean with `true`

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L113

There is no need to check `if (condition == true)`, but simply `if (condition)` will suffice. 

Saves **3 gas** per instance.
