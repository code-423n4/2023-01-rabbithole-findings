## Use of named returns for local variables saves gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:

[File: QuestFactory.sol#L61-L103](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61-L103)

```diff
    function createQuest(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountOrTokenId_,
        string memory contractType_,
        string memory questId_
-    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
+    ) public onlyRole(CREATE_QUEST_ROLE) returns (address newQuestAddr) {

            [ ... ]

-            return address(newQuest);
+            newQuestAddr = address(newQuest);
```
## Avoid comparing boolean expressions to boolean literals
Comparing a boolean value to a boolean literal incurs the `ISZERO` operation and costs more gas than using a boolean expression.

Here are some of the affected code lines that may be refactored as follows:

[File: QuestFactory.sol#L73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73)

```diff
-            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
+            if (!rewardAllowlist[rewardTokenAddress_]) revert RewardNotAllowed();
```
[File: QuestFactory.sol#L221](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221)

```diff
-        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
+        if (quests[questId_].addressMinted[msg.sender]) revert AddressAlreadyMinted();
```