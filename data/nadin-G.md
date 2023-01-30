## Gas Optimizations
Issue 
## [G-01] Multiple accesses of a mapping/array should use a local variable cache
## [G-02] DON’T COMPARE BOOLEAN EXPRESSIONS TO BOOLEAN LITERALS
## [G-03] USE BYTES32 INSTEAD OF STRING
## [G-04] SETTING THE CONSTRUCTOR TO PAYABLE
## [G-05] EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT
## [G-06] STORAGE POINTER TO A STRUCTURE IS CHEAPER THAN COPYING EACH VALUE OF THE STRUCTURE INTO MEMORY, SAME FOR ARRAY AND MAPPING
## [G-07] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS:
## [G-08] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES
## [G-09] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
Total: 55 contexts over 09 issues

## [G-01] Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata
EX: _i = _[i];
No      if (block.timestamp >= _currentSplits[_i].lockedUntil) continue;
Yes    if (block.timestamp >= _currentSplit_i.lockedUntil) continue
Contexts: 02
File: QuestFactory.sol
70: if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L70

File: RabbitHoleReceipt.sol
129: if (tokenIdsForQuest[i] > 0) {
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L129


## [G-02] DON’T COMPARE BOOLEAN EXPRESSIONS TO BOOLEAN LITERALS
Ex: if (<x> == true) => if (<x>), if (<x> == false) => if (!<x>)
Contexts: 02
File: QuestFactory.sol
73: if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
221: if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221


## [G-03] USE BYTES32 INSTEAD OF STRING
Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.
Contexts: 27
File: QuestFactory.sol
28: mapping(string => Quest) public quests;
67: string memory contractType_,
68: string memory questId_
193: function getNumberMinted(string memory questId_) external view returns (uint) {
199: function questInfo(string memory questId_) external view returns (address, uint, uint) {
219: function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L28
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L67
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L68
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L193
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L199
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219

File: RabbitHoleReceipt.sol
30: mapping(uint => string) public questIdForTokenId;
98: function mint(address to_, string memory questId_) public onlyMinter {
110: string memory questId_,
160: ) public view virtual override(ERC721Upgradeable, ERC721URIStorageUpgradeable) returns (string memory) {
164: string memory questId = questIdForTokenId[tokenId_];
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L30
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L98
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L110
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L160
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L164

File: Quest.sol
32:  string memory questId_,
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L32

File: RabbitHoleTickets.sol
102: function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L102

File: Erc20Quest.sol
23:  string memory questId_,
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L23

File: Erc1155Quest.sol
19: string memory questId_,
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L19

File: ReceiptRenderer.sol
23: string memory questId_,
28: ) public view virtual returns (string memory) {
37: return string(abi.encodePacked('data:application/json;base64,', Base64.encode(dataURI)));
42: string memory questId_,
82: function generateAttribute(string memory key, string memory value) public pure returns (string memory) {
93: return string(attribute);
100: function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {
113:  string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L23
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L28
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L37
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L42
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L82
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L93
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L100
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L113

File: TicketRenderer.sol
18: ) public pure returns (string memory) {
30: return string(abi.encodePacked('data:application/json;base64,', Base64.encode(dataURI)));
36:  function generateSVG(uint tokenId_) public pure returns (string memory) {
46: return string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L18
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L30
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L36
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L46


## [G-04] SETTING THE CONSTRUCTOR TO PAYABLE
Saves ~13 gas per instance
Contexts: 06
File: QuestFactory.sol
35: constructor() initializer {}
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L35

File: Quest.sol
26: constructor(
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L26-L34

File: RabbitHoleReceipt.sol
39:    constructor() {
40:        _disableInitializers();
41:    }
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L39-L41

File: RabbitHoleTickets.sol
28: constructor() {
29:        _disableInitializers();
30:    }
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L28

File: Erc20Quest.sol
17: constructor(
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L17-L27

File: Erc1155Quest.sol
13: constructor(
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L13-L21


## [G-05] EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT
Contexts: 01
File: QuestFactory.sol
17: bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L17


## [G-06] STORAGE POINTER TO A STRUCTURE IS CHEAPER THAN COPYING EACH VALUE OF THE STRUCTURE INTO MEMORY, SAME FOR ARRAY AND MAPPING
It may not be obvious, but every time you copy a storage struct/array/mapping to a memory variable, you are literally copying each member by reading it from storage, which is expensive. And when you use the storage keyword, you are just storing a pointer to the storage, which is much cheaper.
Contexts: 11
File: RabbitHoleReceipt.sol
114: uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
125: uint[] memory filteredTokens = new uint[](foundTokens);
164: string memory questId = questIdForTokenId[tokenId_];
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L114
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L125
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L164

File: Quest.sol
99: uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L99

File: ReceiptRenderer.sol
29: bytes memory dataURI = generateDataURI(
48: bytes memory attributes = abi.encodePacked(
63: bytes memory dataURI = abi.encodePacked(
83: bytes memory attribute = abi.encodePacked(
101: bytes memory svg = abi.encodePacked(
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L29
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L48
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L63
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L83
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L101

File: TicketRenderer.sol
19: bytes memory dataURI = abi.encodePacked(
37: bytes memory svg = abi.encodePacked(
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L19
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L37


## [G-07] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS:
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop.
Contexts: 04
File: RabbitHoleReceipt.sol
117: for (uint i = 0; i < msgSenderBalance; i++) {
128: for (uint i = 0; i < msgSenderBalance; i++) {
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128

File: Quest.sol
70: for (uint i = 0; i < tokenIds_.length; i++) {
104: for (uint i = 0; i < tokens.length; i++) {
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104


## [G-08] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES
Contexts: 01
File: Quest.sol
115: redeemedTokens += redeemableTokenCount;
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115


## [G-09] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
It is not necessary to have both a named return and a return statement.
Contexts: 01
File: RabbitHoleTickets.sol
112: ) external view override returns (address receiver, uint256 royaltyAmount) {
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L112
