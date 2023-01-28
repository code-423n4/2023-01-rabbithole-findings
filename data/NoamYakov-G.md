
## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | State variables only set in the constructor should be declared `immutable` | 1 |  2097 |
| [G&#x2011;02] | Don't initialize state variables with default value | 1 |  - |
| [G&#x2011;03] | Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate | 1 |  - |
| [G&#x2011;04] | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops | 10 |  510 |
| [G&#x2011;05] | `internal` functions only called once can be inlined to save gas | 1 |  20 |
| [G&#x2011;06] | Functions guaranteed to revert when called by normal users can be marked `payable` | 2 |  42 |
| [G&#x2011;07] | Optimize names to save gas | 8 |  176 |
| [G&#x2011;08] | Use custom errors rather than `revert()`/`require()` strings to save gas | 3 |  150 |
| [G&#x2011;09] | Multiple accesses of a mapping/array should use a local variable cache | 11 |  462 |
| [G&#x2011;10] | State variables should be cached in stack variables rather than re-reading them from storage | 2 |  194 |
| [G&#x2011;11] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (`-=` too) | 1 |  113 |
| [G&#x2011;12] | `keccak256()` should only need to be called on a specific string literal once | 2 |  84 |
| [G&#x2011;13] | Don't compare boolean expressions to boolean literals | 3 |  27 |

Total: 46 instances over 13 issues with **3875 gas** saved.

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions.

## Gas Optimizations

### [G&#x2011;01]  State variables only set in the constructor should be declared `immutable`
Avoids a Gsset (**20000 gas**) in the constructor, and replaces the first access in each transaction (Gcoldsload - **2100 gas**) and each access thereafter (Gwarmaccess - **100 gas**) with a `PUSH32` (**3 gas**). 

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

*There is 1 instance of this issue:*

```solidity
File: quest-protocol\contracts\Quest.sol

43          questId = questId_;
```

### [G&#x2011;02]  Don't initialize state variables with default value
Avoids a Gsset (**20000 gas**) in the constructor.

*There is 1 instance of this issue:*

```solidity
File: quest-protocol\contracts\Quest.sol

45          redeemedTokens = 0;
```

### [G&#x2011;03]  Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (**20000 gas**) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save **~42 gas per access** due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

*There is 1 instance of this issue:*

```solidity
File: quest-protocol\contracts\RabbitHoleReceipt.sol

30      mapping(uint => string) public questIdForTokenId;
34      mapping(uint => uint) public timestampForTokenId;
```

### [G&#x2011;04]  `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops
The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**.

*There are 10 instances of this issue:*

```solidity
File: quest-protocol\contracts\Quest.sol

70          for (uint i = 0; i < tokenIds_.length; i++) {

104         for (uint i = 0; i < tokens.length; i++) {

/// @audit inside `for`-loop
106                 redeemableTokenCount++;
```

```solidity
File: quest-protocol\contracts\QuestFactory.sol

101             ++questIdCount;

132             ++questIdCount;

226         quests[questId_].numberMinted++;
```

```solidity
File: quest-protocol\contracts\RabbitHoleReceipt.sol

117         for (uint i = 0; i < msgSenderBalance; i++) {

/// @audit inside `for`-loop
121                 foundTokens++;

128         for (uint i = 0; i < msgSenderBalance; i++) {

/// @audit inside `for`-loop
131                 filterTokensIndexTracker++;
```

### [G&#x2011;05]  `internal` functions only called once can be inlined to save gas
Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

*There is 1 instance of this issue:*

```solidity
File: quest-protocol\contracts\QuestFactory.sol

152     function grantDefaultAdminAndCreateQuestRole(address account_) internal {
```

### [G&#x2011;06]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost.

*There are 2 instances of this issue:*

```solidity
File: quest-protocol\contracts\QuestFactory.sol

61      function createQuest(
62          address rewardTokenAddress_,
63          uint256 endTime_,
64          uint256 startTime_,
65          uint256 totalParticipants_,
66          uint256 rewardAmountOrTokenId_,
67          string memory contractType_,
68          string memory questId_
69      ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
```

```solidity
File: quest-protocol\contracts\RabbitHoleTickets.sol

92      function mintBatch(
93          address to_,
94          uint256[] memory ids_,
95          uint256[] memory amounts_,
96          bytes memory data_
97      ) public onlyMinter {
```

### [G&#x2011;07]  Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save **128 gas** each during deployment, and renaming functions to have lower method IDs will save **22 gas** per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

*There are 8 instances of this issue:*

```solidity
File: quest-protocol\contracts\Erc20Quest.sol

/// @audit questFee, protocolFeeRecipient, questFactoryContract,
///     maxTotalRewards(), maxProtocolReward(), start(),
///     withdrawRemainingTokens(), receiptRedeemers(), protocolFee(),
///     withdrawFee()
11  contract Erc20Quest is Quest {
```

```solidity
File: quest-protocol\contracts\Erc1155Quest.sol

/// @audit start(), withdrawRemainingTokens()
11  contract Erc1155Quest is Quest, ERC1155Holder {
```

```solidity
File: quest-protocol\contracts\Quest.sol

/// @audit rabbitHoleReceiptContract, rewardToken, endTime, startTime,
///     totalParticipants, rewardAmountInWeiOrTokenId, hasStarted, isPaused,
///     questId, redeemedTokens, start(), pause(), unPause(), claim(),
///     isClaimed(), getRewardAmount(), getRewardToken(),
///     withdrawRemainingTokens()
12  contract Quest is Ownable, IQuest {
```

```solidity
File: quest-protocol\contracts\QuestFactory.sol

/// @audit CREATE_QUEST_ROLE, claimSignerAddress, protocolFeeRecipient,
///     quests, rabbitholeReceiptContract, rewardAllowlist, questFee,
///     questIdCount, initialize(), createQuest(), changeCreateQuestRole(),
///     setClaimSignerAddress(), setProtocolFeeRecipient(),
///     setRabbitHoleReceiptContract(), setRewardAllowlistAddress(),
///     setQuestFee(), getNumberMinted(), questInfo(), recoverSigner(),
///     mintReceipt()
16  contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory {
```

```solidity
File: quest-protocol\contracts\RabbitHoleReceipt.sol

/// @audit questIdForTokenId, royaltyRecipient, minterAddress, royaltyFee,
///     timestampForTokenId, ReceiptRendererContract, QuestFactoryContract,
///     initialize(), setReceiptRenderer(), setRoyaltyRecipient(),
///     setQuestFactory(), setMinterAddress(), setRoyaltyFee(), mint(),
///     getOwnedTokenIdsOfQuest(), tokenURI(), royaltyInfo(),
///     supportsInterface()
15  contract RabbitHoleReceipt is
16      Initializable,
17      ERC721Upgradeable,
18      ERC721EnumerableUpgradeable,
19      ERC721URIStorageUpgradeable,
20      OwnableUpgradeable,
21      IERC2981Upgradeable
22  {
```

```solidity
File: quest-protocol\contracts\RabbitHoleTickets.sol

/// @audit royaltyRecipient, minterAddress, royaltyFee, TicketRendererContract,
///     initialize(), setTicketRenderer(), setRoyaltyRecipient(),
///     setRoyaltyFee(), setMinterAddress(), mint(), mintBatch(), uri(),
///     royaltyInfo(), supportsInterface()
11  contract RabbitHoleTickets is
12      Initializable,
13      ERC1155Upgradeable,
14      OwnableUpgradeable,
15      ERC1155BurnableUpgradeable,
16      IERC2981Upgradeable
17  {
```

```solidity
File: quest-protocol\contracts\ReceiptRenderer.sol

/// @audit generateTokenURI(), generateDataURI(), generateAttribute(),
///     generateSVG()
10  contract ReceiptRenderer {
```

```solidity
File: quest-protocol\contracts\TicketRenderer.sol

/// @audit generateTokenURI(), generateSVG()
10  contract TicketRenderer {
```

### [G&#x2011;08]  Use custom errors rather than `revert()`/`require()` strings to save gas
Custom errors are available from solidity version 0.8.4. Custom errors save [**~50 gas**](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas.

*There are 3 instances of this issue:*

```solidity
File: quest-protocol\contracts\RabbitHoleReceipt.sol

161         require(_exists(tokenId_), 'ERC721URIStorage: URI query for nonexistent token');

162         require(QuestFactoryContract != IQuestFactory(address(0)), 'QuestFactory not set');

182         require(_exists(tokenId_), 'Nonexistent token');
```

### [G&#x2011;09]  Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the key's keccak256 hash (Gkeccak256 - **30 gas**) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata.

*There are 11 instances of this issue:*

```solidity
File: quest-protocol\contracts\QuestFactory.sol

/// @audit `quests[questId_]` on line 70
98              quests[questId_].questAddress = address(newQuest);

/// @audit `quests[questId_]` on line 70
99              quests[questId_].totalParticipants = totalParticipants_;

/// @audit `quests[questId_]` on line 70
129             quests[questId_].questAddress = address(newQuest);

/// @audit `quests[questId_]` on line 70
130             quests[questId_].totalParticipants = totalParticipants_;

/// @audit `quests[questId_]` on line 201
202             quests[questId_].totalParticipants,

/// @audit `quests[questId_]` on line 201
203             quests[questId_].numberMinted

/// @audit `quests[questId_]` on line 220
220         if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();

/// @audit `quests[questId_]` on line 220
221         if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();

/// @audit `quests[questId_]` on line 220
225         quests[questId_].addressMinted[msg.sender] = true;

/// @audit `quests[questId_]` on line 220
226         quests[questId_].numberMinted++;
```

```solidity
File: quest-protocol\contracts\RabbitHoleReceipt.sol

/// @audit `tokenIdsForQuest[i]` on line 128
130                 filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
```

### [G&#x2011;10]  State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (**100 gas**) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*There are 2 instances of this issue:*

```solidity
File: quest-protocol\contracts\QuestFactory.sol

/// @audit `quests[questId_].numberMinted` on line 220
226         quests[questId_].numberMinted++;
```

```solidity
File: quest-protocol\contracts\RabbitHoleReceipt.sol

/// @audit `QuestFactoryContract` on line 162
165         (address questAddress, uint totalParticipants, ) = QuestFactoryContract.questInfo(questId);
```

### [G&#x2011;11]  `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (`-=` too)
Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**. Subtructions act the same way.

*There is 1 instance of this issue:*

```solidity
File: quest-protocol\contracts\Quest.sol

115         redeemedTokens += redeemableTokenCount;
```

### [G&#x2011;12]  `keccak256()` should only need to be called on a specific string literal once
It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to `bytes4` should also only be done once.

*There are 2 instances of this issue:*

```solidity
File: quest-protocol\contracts\QuestFactory.sol

72          if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {

105         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
```

### [G&#x2011;13]  Don't compare boolean expressions to boolean literals
`if (<x> == true)` => `if (<x>)`, `if (<x> == false)` => `if (!<x>)`.

*There are 3 instances of this issue:*

```solidity
File: quest-protocol\contracts\Quest.sol

136         return claimedList[tokenId_] == true;
```

```solidity
File: quest-protocol\contracts\QuestFactory.sol

73              if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

221         if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```
