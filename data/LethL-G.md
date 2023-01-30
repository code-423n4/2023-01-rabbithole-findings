## Findings Summary

|  | Issue | Instances | 
|---|---|---|
| Gas-1 | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 1 |
| Gas-2 | Boolean Equality | 3 |
| Gas-3 | Use of Named Returns for Local Variables Saves Gas | 1 | 
| Gas-4 | Using a ternary operator instead of an "if-else" statement | 1 |
| Gas-5 | Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant | 1 |
| Gas-6 | Pre-calculate the hardcoded hash with `keccak256()` before and only use the result to save gas | 3 |
| Gas-7 | `internal` functions only called once can be inlined to save gas | 1 |
| Gas-8 | Before some functions, we should check some variables for possible gas save | 5 |

### [Gas-1] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using `x = x+y` instead of `x += y` can help to save gas.

Instances(1):
```
File: Quest.sol

115:        redeemedTokens += redeemableTokenCount;
```
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115

### [Gas-2] Boolean Equality
Boolean constants can be used directly and do not need to be compared to true or false. Thus, this will cost more gas usage.
Source: https://github.com/crytic/slither/wiki/Detector-Documentation#boolean-equality

Instances(3):
```
File: Quest.sol

136:        return claimedList[tokenId_] == true;
```
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L135
```
File: QuestFactory.sol

73:        if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

221:        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221

### [Gas-3] Use of Named Returns for Local Variables Saves Gas
Using named return values as temporary local variables can help you conserve gas.
For example, the following code block can be refactored as follows:
```
File: RabbitHoleReceipt.sol

112:    ) public view returns (uint[] memory filteredTokens) {

125:        filteredTokens = new uint[](foundTokens);
```

Instances(1):
```
File: RabbitHoleReceipt.sol

112:    ) public view returns (uint[] memory) {

125:        uint[] memory filteredTokens = new uint[](foundTokens);
```
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L112
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L125

### [Gas-4] Using a ternary operator instead of an "if-else" statement
Replacing an "if-else" statement with a ternary operator can help you save gas.

Instances(1):
```
File: QuestFactory.sol

143:        if (canCreateQuest_) {
144:            _grantRole(CREATE_QUEST_ROLE, account_);
145:        } else {
146:            _revokeRole(CREATE_QUEST_ROLE, account_);
147:        }
```
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L143-L147

### [Gas-5] Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant
The consequence of this is that the `keccak256()` operation is carried out each time the variable is utilized, leading to a rise in gas costs compared to just storing the calculated hash. By making it immutable, the hashing will only be done during contract deployment, which will reduce gas consumption.

Instances(1):
```
File: QuestFactory.sol

17:    bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
```
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L17

### [Gas-6] Pre-calculate the hardcoded hash with `keccak256()` before and only use the result to save gas
Performing the hardcoded `keccak256()` hash calculation every time the contract is deployed or a function is called is less gas-efficient than pre-calculating the hash beforehand and using only its result.

Instances(3):
```
File: QuestFactory.sol

17:    bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');

72:        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {

105:        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
```
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L17
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L105

### [Gas-7] `internal` functions only called once can be inlined to save gas
Not inlining a function incurs an additional cost of 20 to 40 gas due to the requirement of two extra JUMP instructions and additional stack operations for function calls.

Instances(1):
```
File: QuestFactory.sol

152:    function grantDefaultAdminAndCreateQuestRole(address account_) internal {
```
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152

### [Gas-8] Before some functions, we should check some variables for possible gas save
Before making a transfer, it is advisable to check if the amount is 0 to prevent the function from running and incurring gas costs unnecessarily when no transfer will occur.

Instances(5):
```
File: Erc20Quest.sol

60:        IERC20(rewardToken).safeTransfer(msg.sender, amount_); // check amount_

86:        IERC20(rewardToken).safeTransfer(to_, nonClaimableTokens); // check nonClaimableTokens

103:        IERC20(rewardToken).safeTransfer(protocolFeeRecipient, protocolFee()); // check protocolFee()
```

Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L60
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L86
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L103

```
File: Erc1155Quest.sol

42:        IERC1155(rewardToken).safeTransferFrom(address(this), msg.sender, rewardAmountInWeiOrTokenId, amount_, '0x00'); // check amount_

60:        IERC1155(rewardToken).balanceOf(address(this), rewardAmountInWeiOrTokenId),
```

Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L42
Link to Code: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L60