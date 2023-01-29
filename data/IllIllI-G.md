
## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Shorten the array rather than copying to a new one | 1 |  - |
| [G&#x2011;02] | Hash shouldn't be re-calculated on every iteration of the `for`-loop | 1 |  - |
| [G&#x2011;03] | Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas | 12 |  1440 |
| [G&#x2011;04] | Multiple accesses of a mapping/array should use a local variable cache | 9 |  378 |
| [G&#x2011;05] | The result of function calls should be cached rather than re-calling the function | 1 |  - |
| [G&#x2011;06] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 1 |  113 |
| [G&#x2011;07] | `internal` functions only called once can be inlined to save gas | 1 |  20 |
| [G&#x2011;08] | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops | 4 |  240 |
| [G&#x2011;09] | Optimize names to save gas | 8 |  176 |
| [G&#x2011;10] | String literals passed to `abi.encode()`/`abi.encodePacked()` should not be split by commas | 5 |  - |
| [G&#x2011;11] | Using `private` rather than `public` for constants, saves gas | 5 |  - |
| [G&#x2011;12] | Don't compare boolean expressions to boolean literals | 3 |  27 |
| [G&#x2011;13] | Use custom errors rather than `revert()`/`require()` strings to save gas | 3 |  - |
| [G&#x2011;14] | Functions guaranteed to revert when called by normal users can be marked `payable` | 2 |  42 |

Total: 56 instances over 14 issues with **2436 gas** saved

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations

### [G&#x2011;01]  Shorten the array rather than copying to a new one
Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

*There is 1 instance of this issue:*

```solidity
File: /contracts/RabbitHoleReceipt.sol

125          uint[] memory filteredTokens = new uint[](foundTokens);
126          uint filterTokensIndexTracker = 0;
127  
128          for (uint i = 0; i < msgSenderBalance; i++) {
129              if (tokenIdsForQuest[i] > 0) {
130                  filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
131                  filterTokensIndexTracker++;
132              }
133:         }

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L125-L133

### [G&#x2011;02]  Hash shouldn't be re-calculated on every iteration of the `for`-loop
Calculate the hash outside of the loop, and use that value within the loop

*There is 1 instance of this issue:*

```solidity
File: /contracts/RabbitHoleReceipt.sol

119:             if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L119

### [G&#x2011;03]  Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas
When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. **Each iteration of this for-loop costs at least 60 gas** (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it's still valid for implementation contracs to use `calldata` arguments instead. 

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gass-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I've also flagged instances where the function is `public` but can be marked as `external` since it's not called by the contract, and cases where a constructor is involved

*There are 12 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

/// @audit contractType_
/// @audit questId_
61        function createQuest(
62            address rewardTokenAddress_,
63            uint256 endTime_,
64            uint256 startTime_,
65            uint256 totalParticipants_,
66            uint256 rewardAmountOrTokenId_,
67            string memory contractType_,
68            string memory questId_
69:       ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {

/// @audit questId_
193:      function getNumberMinted(string memory questId_) external view returns (uint) {

/// @audit questId_
199:      function questInfo(string memory questId_) external view returns (address, uint, uint) {

/// @audit questId_
/// @audit signature_
219:      function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61-L69

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit questId_
98:       function mint(address to_, string memory questId_) public onlyMinter {

/// @audit questId_
109       function getOwnedTokenIdsOfQuest(
110           string memory questId_,
111           address claimingAddress_
112:      ) public view returns (uint[] memory) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L98

```solidity
File: contracts/RabbitHoleTickets.sol

/// @audit data_
83:       function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {

/// @audit ids_
/// @audit amounts_
/// @audit data_
92        function mintBatch(
93            address to_,
94            uint256[] memory ids_,
95            uint256[] memory amounts_,
96            bytes memory data_
97:       ) public onlyMinter {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L83

### [G&#x2011;04]  Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the key's keccak256 hash (Gkeccak256 - **30 gas**) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata

*There are 9 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

/// @audit quests[questId_] on line 70
98:               quests[questId_].questAddress = address(newQuest);

/// @audit quests[questId_] on line 98
99:               quests[questId_].totalParticipants = totalParticipants_;

/// @audit quests[questId_] on line 129
130:              quests[questId_].totalParticipants = totalParticipants_;

/// @audit quests[questId_] on line 201
202:              quests[questId_].totalParticipants,

/// @audit quests[questId_] on line 202
203:              quests[questId_].numberMinted

/// @audit quests[questId_] on line 220
220:          if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();

/// @audit quests[questId_] on line 220
221:          if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();

/// @audit quests[questId_] on line 221
225:          quests[questId_].addressMinted[msg.sender] = true;

/// @audit quests[questId_] on line 225
226:          quests[questId_].numberMinted++;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L98

### [G&#x2011;05]  The result of function calls should be cached rather than re-calling the function
The instances below point to the second+ call of the function within a single function

*There is 1 instance of this issue:*

```solidity
File: contracts/ReceiptRenderer.sol

/// @audit tokenId_.toString() on line 52
66:               tokenId_.toString(),

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L66

### [G&#x2011;06]  `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

*There is 1 instance of this issue:*

```solidity
File: contracts/Quest.sol

115:          redeemedTokens += redeemableTokenCount;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115

### [G&#x2011;07]  `internal` functions only called once can be inlined to save gas
Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

*There is 1 instance of this issue:*

```solidity
File: contracts/QuestFactory.sol

152:      function grantDefaultAdminAndCreateQuestRole(address account_) internal {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152

### [G&#x2011;08]  `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops
The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**

*There are 4 instances of this issue:*

```solidity
File: contracts/Quest.sol

70:           for (uint i = 0; i < tokenIds_.length; i++) {

104:          for (uint i = 0; i < tokens.length; i++) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70

```solidity
File: contracts/RabbitHoleReceipt.sol

117:          for (uint i = 0; i < msgSenderBalance; i++) {

128:          for (uint i = 0; i < msgSenderBalance; i++) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117

### [G&#x2011;09]  Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save **128 gas** each during deployment, and renaming functions to have lower method IDs will save **22 gas** per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

*There are 8 instances of this issue:*

```solidity
File: contracts/Erc20Quest.sol

/// @audit maxTotalRewards(), maxProtocolReward(), receiptRedeemers(), protocolFee(), withdrawFee()
11:   contract Erc20Quest is Quest {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L11

```solidity
File: contracts/interfaces/IQuest.sol

/// @audit isClaimed(), getRewardAmount(), getRewardToken()
6:    interface IQuest {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L6

```solidity
File: contracts/QuestFactory.sol

/// @audit initialize(), createQuest(), changeCreateQuestRole(), setClaimSignerAddress(), setProtocolFeeRecipient(), setRabbitHoleReceiptContract(), setRewardAllowlistAddress(), setQuestFee(), getNumberMinted(), questInfo(), recoverSigner(), mintReceipt()
16:   contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L16

```solidity
File: contracts/Quest.sol

/// @audit unPause(), claim(), isClaimed(), getRewardAmount(), getRewardToken(), withdrawRemainingTokens()
12:   contract Quest is Ownable, IQuest {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L12

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit initialize(), setReceiptRenderer(), setRoyaltyRecipient(), setQuestFactory(), setMinterAddress(), setRoyaltyFee(), mint(), getOwnedTokenIdsOfQuest()
15:   contract RabbitHoleReceipt is

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L15

```solidity
File: contracts/RabbitHoleTickets.sol

/// @audit initialize(), setTicketRenderer(), setRoyaltyRecipient(), setRoyaltyFee(), setMinterAddress(), mint(), mintBatch()
11:   contract RabbitHoleTickets is

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L11

```solidity
File: contracts/ReceiptRenderer.sol

/// @audit generateTokenURI(), generateDataURI(), generateAttribute(), generateSVG()
10:   contract ReceiptRenderer {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L10

```solidity
File: contracts/TicketRenderer.sol

/// @audit generateTokenURI(), generateSVG()
10:   contract TicketRenderer {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L10

### [G&#x2011;10]  String literals passed to `abi.encode()`/`abi.encodePacked()` should not be split by commas
String literals can be split into multiple parts and still be considered as a single string literal. Adding commas between each chunk makes it no longer a single string, and instead multiple strings. EACH new comma costs [***21 gas***](https://gist.github.com/IllIllI000/837d1b36c16c9bfe1010f9f775a09bbf) due to stack operations and separate `MSTORE`s

*There are 5 instances of this issue:*

```solidity
File: contracts/ReceiptRenderer.sol

/// @audit 4 commas
63            bytes memory dataURI = abi.encodePacked(
64                '{',
65                '"name": "RabbitHole.gg Receipt #',
66                tokenId_.toString(),
67                '",',
68                '"description": "RabbitHole.gg Receipts are used to claim rewards from completed quests.",',
69                '"image": "',
70                generateSVG(tokenId_, questId_),
71                '",',
72                '"attributes": ',
73                attributes,
74                '}'
75:           );

/// @audit 3 commas
83            bytes memory attribute = abi.encodePacked(
84                '{',
85                '"trait_type": "',
86                key,
87                '",',
88                '"value": "',
89                value,
90                '"',
91                '}'
92:           );

/// @audit 5 commas
101           bytes memory svg = abi.encodePacked(
102               '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
103               '<style>.base { fill: white; font-family: serif; font-size: 14px; }</style>',
104               '<rect width="100%" height="100%" fill="black" />',
105               '<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest #',
106               questId_,
107               '</text>',
108               '<text x="70%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #',
109               tokenId_,
110               '</text>',
111               '</svg>'
112:          );

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L63-L75

```solidity
File: contracts/TicketRenderer.sol

/// @audit 4 commas
19            bytes memory dataURI = abi.encodePacked(
20                '{',
21                '"name": "RabbitHole Tickets #',
22                tokenId_.toString(),
23                '",',
24                '"description": "A reward for completing quests within RabbitHole, with unk(no)wn utility",',
25                '"image": "',
26                generateSVG(tokenId_),
27                '"',
28                '}'
29:           );

/// @audit 4 commas
37            bytes memory svg = abi.encodePacked(
38                '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
39                '<style>.base { fill: white; font-family: serif; font-size: 14px; }</style>',
40                '<rect width="100%" height="100%" fill="black" />',
41                '<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Tickets #',
42                tokenId_.toString(),
43                '</text>',
44                '</svg>'
45:           );

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L19-L29

### [G&#x2011;11]  Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There are 5 instances of this issue:*

```solidity
File: contracts/Erc20Quest.sol

13:       uint256 public immutable questFee;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L13

```solidity
File: contracts/Quest.sol

15:       uint256 public immutable endTime;

16:       uint256 public immutable startTime;

17:       uint256 public immutable totalParticipants;

18:       uint256 public immutable rewardAmountInWeiOrTokenId;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L15

### [G&#x2011;12]  Don't compare boolean expressions to boolean literals
`if (<x> == true)` => `if (<x>)`, `if (<x> == false)` => `if (!<x>)`

*There are 3 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

73:               if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

221:          if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73

```solidity
File: contracts/Quest.sol

136:          return claimedList[tokenId_] == true;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L136

### [G&#x2011;13]  Use custom errors rather than `revert()`/`require()` strings to save gas
Custom errors are available from solidity version 0.8.4. Custom errors save [**~50 gas**](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas

*There are 3 instances of this issue:*

```solidity
File: contracts/RabbitHoleReceipt.sol

161:          require(_exists(tokenId_), 'ERC721URIStorage: URI query for nonexistent token');

162:          require(QuestFactoryContract != IQuestFactory(address(0)), 'QuestFactory not set');

182:          require(_exists(tokenId_), 'Nonexistent token');

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L161

### [G&#x2011;14]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

*There are 2 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

61        function createQuest(
62            address rewardTokenAddress_,
63            uint256 endTime_,
64            uint256 startTime_,
65            uint256 totalParticipants_,
66            uint256 rewardAmountOrTokenId_,
67            string memory contractType_,
68            string memory questId_
69:       ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61-L69

```solidity
File: contracts/RabbitHoleTickets.sol

92        function mintBatch(
93            address to_,
94            uint256[] memory ids_,
95            uint256[] memory amounts_,
96            bytes memory data_
97:       ) public onlyMinter {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L92-L97


___

## Excluded findings
These findings are excluded from awards calculations because there are publicly-available automated tools that find them. The valid ones appear here for completeness

## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | `<array>.length` should not be looked up in every loop of a `for`-loop | 2 |  6 |
| [G&#x2011;02] | `require()`/`revert()` strings longer than 32 bytes cost extra gas | 1 |  - |
| [G&#x2011;03] | Using `bool`s for storage incurs overhead | 4 |  68400 |
| [G&#x2011;04] | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 8 |  40 |
| [G&#x2011;05] | Using `private` rather than `public` for constants, saves gas | 1 |  - |
| [G&#x2011;06] | Functions guaranteed to revert when called by normal users can be marked `payable` | 25 |  525 |

Total: 41 instances over 6 issues with **68971 gas** saved

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations

### [G&#x2011;01]  `<array>.length` should not be looked up in every loop of a `for`-loop
The overheads outlined below are _PER LOOP_, excluding the first loop
* storage arrays incur a Gwarmaccess (**100 gas**)
* memory arrays use `MLOAD` (**3 gas**)
* calldata arrays use `CALLDATALOAD` (**3 gas**)

Caching the length changes each of these to a `DUP<N>` (**3 gas**), and gets rid of the extra `DUP<N>` needed to store the stack offset

*There are 2 instances of this issue:*

```solidity
File: contracts/Quest.sol

/// @audit (valid but excluded finding)
70:           for (uint i = 0; i < tokenIds_.length; i++) {

/// @audit (valid but excluded finding)
104:          for (uint i = 0; i < tokens.length; i++) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70

### [G&#x2011;02]  `require()`/`revert()` strings longer than 32 bytes cost extra gas
Each extra memory word of bytes past the original 32 [incurs an MSTORE](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#consider-having-short-revert-strings) which costs **3 gas**

*There is 1 instance of this issue:*

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit (valid but excluded finding)
161:          require(_exists(tokenId_), 'ERC721URIStorage: URI query for nonexistent token');

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L161

### [G&#x2011;03]  Using `bool`s for storage incurs overhead
```solidity
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27
Use `uint256(1)` and `uint256(2)` for true/false to avoid a Gwarmaccess (**[100 gas](https://gist.github.com/IllIllI000/1b70014db712f8572a72378321250058)**) for the extra SLOAD, and to avoid Gsset (**20000 gas**) when changing from `false` to `true`, after having been `true` in the past

*There are 4 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

/// @audit (valid but excluded finding)
30:       mapping(address => bool) public rewardAllowlist;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L30

```solidity
File: contracts/Quest.sol

/// @audit (valid but excluded finding)
19:       bool public hasStarted;

/// @audit (valid but excluded finding)
20:       bool public isPaused;

/// @audit (valid but excluded finding)
24:       mapping(uint256 => bool) private claimedList;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L19

### [G&#x2011;04]  `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
Saves **5 gas per loop**

*There are 8 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

/// @audit (valid but excluded finding)
226:          quests[questId_].numberMinted++;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L226

```solidity
File: contracts/Quest.sol

/// @audit (valid but excluded finding)
70:           for (uint i = 0; i < tokenIds_.length; i++) {

/// @audit (valid but excluded finding)
104:          for (uint i = 0; i < tokens.length; i++) {

/// @audit (valid but excluded finding)
106:                  redeemableTokenCount++;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit (valid but excluded finding)
117:          for (uint i = 0; i < msgSenderBalance; i++) {

/// @audit (valid but excluded finding)
121:                  foundTokens++;

/// @audit (valid but excluded finding)
128:          for (uint i = 0; i < msgSenderBalance; i++) {

/// @audit (valid but excluded finding)
131:                  filterTokensIndexTracker++;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117

### [G&#x2011;05]  Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There is 1 instance of this issue:*

```solidity
File: contracts/QuestFactory.sol

/// @audit (valid but excluded finding)
17:       bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L17

### [G&#x2011;06]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

*There are 25 instances of this issue:*

```solidity
File: contracts/Erc1155Quest.sol

/// @audit (valid but excluded finding)
54:       function withdrawRemainingTokens(address to_) public override onlyOwner {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54

```solidity
File: contracts/Erc20Quest.sol

/// @audit (valid but excluded finding)
81:       function withdrawRemainingTokens(address to_) public override onlyOwner {

/// @audit (valid but excluded finding)
102:      function withdrawFee() public onlyAdminWithdrawAfterEnd {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81

```solidity
File: contracts/QuestFactory.sol

/// @audit (valid but excluded finding)
142:      function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {

/// @audit (valid but excluded finding)
159:      function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {

/// @audit (valid but excluded finding)
165:      function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {

/// @audit (valid but excluded finding)
172:      function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {

/// @audit (valid but excluded finding)
179:      function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {

/// @audit (valid but excluded finding)
186:      function setQuestFee(uint256 questFee_) public onlyOwner {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L142

```solidity
File: contracts/Quest.sol

/// @audit (valid but excluded finding)
50:       function start() public virtual onlyOwner {

/// @audit (valid but excluded finding)
57:       function pause() public onlyOwner onlyStarted {

/// @audit (valid but excluded finding)
63:       function unPause() public onlyOwner onlyStarted {

/// @audit (valid but excluded finding)
96:       function claim() public virtual onlyQuestActive {

/// @audit (valid but excluded finding)
150:      function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit (valid but excluded finding)
65:       function setReceiptRenderer(address receiptRenderer_) public onlyOwner {

/// @audit (valid but excluded finding)
71:       function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {

/// @audit (valid but excluded finding)
77:       function setQuestFactory(address questFactory_) public onlyOwner {

/// @audit (valid but excluded finding)
83:       function setMinterAddress(address minterAddress_) public onlyOwner {

/// @audit (valid but excluded finding)
90:       function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {

/// @audit (valid but excluded finding)
98:       function mint(address to_, string memory questId_) public onlyMinter {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L65

```solidity
File: contracts/RabbitHoleTickets.sol

/// @audit (valid but excluded finding)
54:       function setTicketRenderer(address ticketRenderer_) public onlyOwner {

/// @audit (valid but excluded finding)
60:       function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {

/// @audit (valid but excluded finding)
66:       function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {

/// @audit (valid but excluded finding)
73:       function setMinterAddress(address minterAddress_) public onlyOwner {

/// @audit (valid but excluded finding)
83:       function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L54
