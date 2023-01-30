# Summary

## Gas Optimizations

|        | Issue                                                                                                                      | Instances |
| ------ | :------------------------------------------------------------------------------------------------------------------------- | :-------: |
| GAS-1  | `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables                      |     1     |
| GAS-2  | Don’t compare boolean expressions to boolean literals                                                                      |     2     |
| GAS-3  | Setting the constructor to payable                                                                                         |     3     |
| GAS-4  | Internal functions only called once can be inlined to save gas                                                             |     3     |
| GAS-5  | Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate |     1     |
| GAS-6  | Optimize names to save gas                                                                                                 |    10     |
| GAS-7  | The increment in for loop postcondition can be made unchecked                                                              |     4     |
| GAS-8  | Functions not used internally could be marked external                                                                     |    21     |
| GAS-9  | Using storage instead of memory for structs/arrays saves gas                                                               |     2     |
| GAS-10 | Use bitmaps to save gas                                                                                                    |     1     |

### [GAS-1] `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables

Using compound assignment operators for state variables (like `State += X` or S`tate -= X` …) it’s more expensive than using operator assignment (like `State = State + X` or `State = State - X` …).

##### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/Quest.sol

115:         redeemedTokens += redeemableTokenCount;

```

### [GAS-2] Don’t compare boolean expressions to boolean literals

#### Description:

`if (<x> == true) => if (<x>)`, `if (<x> == false) => if (!<x>)`

##### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/QuestFactory.sol

73:             if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

221:         if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();

```

[QuestFactory.sol#L73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73)

[QuestFactory.sol#L221](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221)

### [GAS-3] Setting the constructor to payable

Saves ~13 gas per instance

##### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/Erc1155Quest.sol
13:      constructor(
14:        address rewardTokenAddress_,
15:        uint256 endTime_,
16:        uint256 startTime_,
17:        uint256 totalParticipants_,
18:        uint256 rewardAmountInWeiOrTokenId_,
19:        string memory questId_,
20:        address receiptContractAddress_
21:    )

```

[Erc1155Quest.sol#L13](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L13)

```solidity
File: quest-protocol/contracts/Erc20Quest.sol
17:    constructor(
18:        address rewardTokenAddress_,
19:        uint256 endTime_,
20:        uint256 startTime_,
21:        uint256 totalParticipants_,
22:        uint256 rewardAmountInWeiOrTokenId_,
23:        string memory questId_,
24:        address receiptContractAddress_,
25:        uint256 questFee_,
26:        address protocolFeeRecipient_
27:    )

```

[Erc20Quest.sol#17](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L17)

```solidity
File: quest-protocol/contracts/Quest.sol
26:    constructor(
27:        address rewardTokenAddress_,
28:        uint256 endTime_,
29:        uint256 startTime_,
30:        uint256 totalParticipants_,
31:        uint256 rewardAmountInWeiOrTokenId_,
32:        string memory questId_,
33:        address receiptContractAddress_
34:    ) {

```

[Quest.sol#L26](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L26)

### [GAS-4] Internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

##### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/Quest.sol

122:     function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {

129:     function _transferRewards(uint256 amount_) internal virtual {

```

[Quest.sol#L122](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L122)

[Quest.sol#L129](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L129)

```solidity
File: quest-protocol/contracts/QuestFactory.sol

152:     function grantDefaultAdminAndCreateQuestRole(address account_) internal {

```

[QuestFactory.sol#L152](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152)

### [GAS-5] Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

##### **Proof Of Concept**

- `questIdForTokenId` and `timestampForTokenId` in `mint` function,

```solidity
File: quest-protocol/contracts/RabbitHoleReceipt.sol

30:     mapping(uint => string) public questIdForTokenId;

34:     mapping(uint => uint) public timestampForTokenId;

```

[RabbitHoleReceipt.sol#L30](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L30)

[RabbitHoleReceipt.sol#L34](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L34)

Both of them are being used in the same function mostly consider making them a structs instead.

### [GAS-6] Optimize names to save gas

`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. In this report are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

_There are 10 instances of this issue._

### [GAS-7] The increment in for loop postcondition can be made unchecked

This is only relevant if you are using the default solidity checked arithmetic.

The for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of `i` is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of `i` to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks. One can manually do this.

##### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/Quest.sol

70:         for (uint i = 0; i < tokenIds_.length; i++) {

104:         for (uint i = 0; i < tokens.length; i++) {

```

[Quest.sol#L70](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70)

[Quest.sol#L104](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104)

```solidity
File: quest-protocol/contracts/RabbitHoleReceipt.sol

117:         for (uint i = 0; i < msgSenderBalance; i++) {

128:         for (uint i = 0; i < msgSenderBalance; i++) {

```

[RabbitHoleReceipt.sol#L117](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117)

[RabbitHoleReceipt.sol#L128](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128)

### [GAS-8] Functions not used internally could be marked external

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

##### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/Erc20Quest.sol

102:     function withdrawFee() public onlyAdminWithdrawAfterEnd {

```

```solidity
File: quest-protocol/contracts/Quest.sol

140:     function getRewardAmount() public view returns (uint256) {

145:     function getRewardToken() public view returns (address) {

150:     function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}

```

```solidity
File: quest-protocol/contracts/QuestFactory.sol

142:     function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {

159:     function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {

172:     function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {

179:     function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {

219:     function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {

```

```solidity
File: quest-protocol/contracts/RabbitHoleReceipt.sol

65:     function setReceiptRenderer(address receiptRenderer_) public onlyOwner {

71:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {

77:     function setQuestFactory(address questFactory_) public onlyOwner {

83:     function setMinterAddress(address minterAddress_) public onlyOwner {

90:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {

```

```solidity
File: quest-protocol/contracts/RabbitHoleTickets.sol

54:     function setTicketRenderer(address ticketRenderer_) public onlyOwner {

60:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {

66:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {

73:     function setMinterAddress(address minterAddress_) public onlyOwner {

83:     function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {

102:     function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {

```

```solidity
File: quest-protocol/contracts/TicketRenderer.sol

16:     function generateTokenURI(uint tokenId_) public pure returns (string memory) {

```

## [GAS-9] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

##### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/Quest.sol

69:     function _setClaimed(uint256[] memory tokenIds_) private {

```

```solidity
File: quest-protocol/contracts/RabbitHoleReceipt.sol

114:         uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);

```

### [GAS-10] Use bitmaps to save gas

During calls to `claimedList` the executed mapping is updated, setting the bool flag to true, but because the way the EVM works it allocates an entire storage slot (256 bits) every time a new flag is set. SSTORE opcode cost up to 20000 gas for uninitialized slots like these. Consider using bitmaps instead, this enables you to convert the `mapping(uint256 => bool)` to a `mapping(uint256 => uint256)` and pay the cost of slot initialization just once every 256 nonces added, so the high gas costs are amortized over many calls.

##### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/Quest.sol

24:     mapping(uint256 => bool) private claimedList;

```

[Quest.sol#L24](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L24)

Consider changing `mapping(uint256 => bool) private claimedList;` to `mapping(uint256 => uint256) private claimedList;`.
