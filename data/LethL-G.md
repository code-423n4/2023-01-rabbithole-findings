## Findings Summary

|  | Issue | Instances | 
|---|---|---|
| Gas-1 | `x += y` costs more gas than `x = x + y` for state variables | 1 |
| Gas-2 | Boolean Equality | 3 |
| Gas-3 | Use of Named Returns for Local Variables Saves Gas | 1 | 
| Gas-4 | Using a ternary operator instead of "if-else" statement | 1 |

### [Gas-1] `x += y` costs more gas than `x = x + y` for state variables
Using `x = x+y` instead of `x += y` can help to save gas. The `x = x+y` operator creates a new memory location for the updated value of `x`, whereas the `x += y` operator directly updates the value of `x` in the existing memory location. This can help to minimize the amount of gas needed to execute the contract.

Instances(1):
```
File: Quest.sol

115:        redeemedTokens += redeemableTokenCount;
```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115)

### [Gas-2] Boolean Equality
Boolean constants can be used directly and do not need to be compared to true or false. Thus, this will cost more gas usage. Source: [Boolean Equality](https://github.com/crytic/slither/wiki/Detector-Documentation#boolean-equality)

Instances(3):
```
File: Quest.sol

136:        return claimedList[tokenId_] == true;
```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L135)
```
File: QuestFactory.sol

73:        if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
221:        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)

### [Gas-3] Use of Named Returns for Local Variables Saves Gas
Using named return values as temporary local variables can help you conserve gas.
For example, the following code block can be refactored as follows:
```
File: RabbitHoleReceipt.sol

112:    ) public view returns (uint[] memory filteredTokens) {
...
125:        filteredTokens = new uint[](foundTokens);
```

Instances(1):
```
File: RabbitHoleReceipt.sol

112:    ) public view returns (uint[] memory) {
...
125:        uint[] memory filteredTokens = new uint[](foundTokens);
```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol)

### [Gas-4] Using a ternary operator instead of "if-else" statement
Replacing an "if-else" statement with a ternary operator can help you save gas.
For example, the following code block can be refactored as follows:
```
File: QuestFactory.sol

142:    function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
143:        canCreateQuest ? _grantRole(CREATE_QUEST_ROLE, account_) : _revokeRole(CREATE_QUEST_ROLE, account_);
144:    }
```

Instances(1):
```
File: QuestFactory.sol

142:    function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
143:        if (canCreateQuest_) {
144:            _grantRole(CREATE_QUEST_ROLE, account_);
145:        } else {
146:            _revokeRole(CREATE_QUEST_ROLE, account_);
147:        }
148:    }
```
[Link to Code](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L143-L147)