# Gas Optimizations Report

|         | Issue                                                                                                                              | Instances |
| ------- | :--------------------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-001] | x += y or x -= y costs more gas than x = x + y or x = x - y for state variables                                                    |     1     |
| [G-002] | `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping` |     3     |
| [G-003] | Functions guaranteed to revert when called by normal users can be marked payable                                                   |    25     |
| [G-004] | Multiple accesses of a `mapping/array` should use a local variable cache                                                           |     4     |
| [G-005] | internal functions only called once can be inlined to save gas                                                                     |     3     |
| [G-006] | Replace modifier with function                                                                                                     |     5     |

## [G-001] x += y or x -= y costs more gas than x = x + y or x = x - y for state variables

### Impact

Using the addition operator instead of plus-equals saves 113 gas. Usually does not work with struct and mappings.

### Findings

Total:1

[contracts/Quest.sol#L115](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115)

```solidity
115:    redeemedTokens += redeemableTokenCount;
```

## [G-002] `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping`

### Impact

It may not be obvious, but every time you copy a storage `struct/array/mapping` to a `memory` variable, you are literally copying each member by reading it from `storage`, which is expensive. And when you use the `storage` keyword, you are just storing a pointer to the storage, which is much cheaper.

### Findings

Total:3

[contracts/RabbitHoleReceipt.sol#L114](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L114)

```solidity
114:    uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
```

[contracts/RabbitHoleReceipt.sol#L125](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L125)

```solidity
125:    uint[] memory filteredTokens = new uint[](foundTokens);
```

[contracts/Quest.sol#L99](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L99)

```solidity
99:    uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);
```

### Recommendation

Using `storage` instead of `memory`

## [G-003] Functions guaranteed to revert when called by normal users can be marked payable

### Impact

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
The extra opcodes avoided are `CALLVALUE(2)`,`DUP1(3)`,`ISZERO(3)`,`PUSH2(3)`,`JUMPI(10)`,`PUSH1(3)`,`DUP1(3)`,`REVERT(0)`,`JUMPDEST(1)`,`POP(2)`, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Findings

Total:25

[contracts/QuestFactory.sol#L69](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L69)

```solidity
69:    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
```

[contracts/QuestFactory.sol#L179](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L179)

```solidity
179:    function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
```

[contracts/QuestFactory.sol#L159](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159)

```solidity
159:    function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
```

[contracts/QuestFactory.sol#L172](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L172)

```solidity
172:    function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
```

[contracts/QuestFactory.sol#L165](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L165)

```solidity
165:    function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
```

[contracts/QuestFactory.sol#L186](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L186)

```solidity
186:    function setQuestFee(uint256 questFee_) public onlyOwner {
```

[contracts/QuestFactory.sol#L142](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L142)

```solidity
142:    function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
```

[contracts/RabbitHoleReceipt.sol#L77](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L77)

```solidity
77:    function setQuestFactory(address questFactory_) public onlyOwner {
```

[contracts/RabbitHoleReceipt.sol#L90](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90)

```solidity
90:    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
```

[contracts/RabbitHoleReceipt.sol#L83](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L83)

```solidity
83:    function setMinterAddress(address minterAddress_) public onlyOwner {
```

[contracts/RabbitHoleReceipt.sol#L65](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L65)

```solidity
65:    function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
```

[contracts/RabbitHoleReceipt.sol#L71](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71)

```solidity
71:    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
```

[contracts/RabbitHoleReceipt.sol#L98](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L98)

```solidity
98:    function mint(address to_, string memory questId_) public onlyMinter {
```

[contracts/Quest.sol#L150](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150)

```solidity
150:    function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
```

[contracts/Quest.sol#L57](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57)

```solidity
57:    function pause() public onlyOwner onlyStarted {
```

[contracts/Quest.sol#L63](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L63)

```solidity
63:    function unPause() public onlyOwner onlyStarted {
```

[contracts/Quest.sol#L50](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50)

```solidity
50:    function start() public virtual onlyOwner {
```

[contracts/Quest.sol#L96](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96)

```solidity
96:    function claim() public virtual onlyQuestActive {
```

[contracts/RabbitHoleTickets.sol#L54](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L54)

```solidity
54:    function setTicketRenderer(address ticketRenderer_) public onlyOwner {
```

[contracts/RabbitHoleTickets.sol#L66](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66)

```solidity
66:    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
```

[contracts/RabbitHoleTickets.sol#L73](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L73)

```solidity
73:    function setMinterAddress(address minterAddress_) public onlyOwner {
```

[contracts/RabbitHoleTickets.sol#L83](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L83)

```solidity
83:    function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {
```

[contracts/RabbitHoleTickets.sol#L97](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L97)

```solidity
97:    ) public onlyMinter {
```

[contracts/RabbitHoleTickets.sol#L60](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L60)

```solidity
60:    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
```

[contracts/Erc20Quest.sol#L102](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L102)

```solidity
102:    function withdrawFee() public onlyAdminWithdrawAfterEnd {
```

### Recommendation

Mark the function as payable.

## [G-004] Multiple accesses of a `mapping/array` should use a local variable cache

### Impact

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into `memory/calldata` 

### Findings

Total:4

[contracts/QuestFactory.sol#L225](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L225)

```solidity
225:    quests[questId_].addressMinted[msg.sender] = true;
```

[contracts/QuestFactory.sol#L180](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L180)

```solidity
180:    rewardAllowlist[rewardAddress_] = allowed_;
```

[contracts/RabbitHoleReceipt.sol#L101](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L101)

```solidity
101:    questIdForTokenId[newTokenID] = questId_;
```

[contracts/RabbitHoleReceipt.sol#L102](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L102)

```solidity
102:    timestampForTokenId[newTokenID] = block.timestamp;
```

## [G-005] internal functions only called once can be inlined to save gas

### Impact

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

### Findings

Total:3

[contracts/RabbitHoleReceipt.sol#L139](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L139)

```solidity
139:    function _burn(uint256 tokenId_) internal override(ERC721Upgradeable, ERC721URIStorageUpgradeable) {
```

[contracts/Quest.sol#L122](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L122)

```solidity
122:    function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {
```

[contracts/Quest.sol#L129](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L129)

```solidity
129:    function _transferRewards(uint256 amount_) internal virtual {
```

### Recommendation

Same as description

## [G-006] Replace modifier with function

### Impact

Modifiers make code more elegant, but cost more than normal functions.

### Findings

Total:5

[contracts/RabbitHoleReceipt.sol#L58](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L58)

```solidity
58:    modifier onlyMinter() {
```

[contracts/Quest.sol#L88](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L88)

```solidity
88:    modifier onlyQuestActive() {
```

[contracts/Quest.sol#L82](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L82)

```solidity
82:    modifier onlyStarted() {
```

[contracts/Quest.sol#L76](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76)

```solidity
76:    modifier onlyAdminWithdrawAfterEnd() {
```

[contracts/RabbitHoleTickets.sol#L47](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L47)

```solidity
47:    modifier onlyMinter() {
```
