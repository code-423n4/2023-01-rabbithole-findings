## Summary<a name="Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | Comparing to constant boolean | 2 | - |
| [GAS&#x2011;2](#GAS&#x2011;2) | State variables only set in the `constructor` should be declared immutable | 1 | 2100 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Setting the `constructor` to `payable` | 6 | 78 |
| [GAS&#x2011;4](#GAS&#x2011;4) | Empty Blocks Should Be Removed Or Emit Something | 1 | - |
| [GAS&#x2011;5](#GAS&#x2011;5) | Using `delete` statement can save gas | 3 | - |
| [GAS&#x2011;6](#GAS&#x2011;6) | Use hardcoded address instead `address(this)` | 6 | - |
| [GAS&#x2011;7](#GAS&#x2011;7) | `internal` functions only called once can be inlined to save gas | 9 | 198 |
| [GAS&#x2011;8](#GAS&#x2011;8) | Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate | 2 | - |
| [GAS&#x2011;9](#GAS&#x2011;9) | Optimize names to save gas | 6 | 132 |
| [GAS&#x2011;10](#GAS&#x2011;10) | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 1 | - |
| [GAS&#x2011;11](#GAS&#x2011;11) | Public Functions To External | 52 | - |
| [GAS&#x2011;12](#GAS&#x2011;12) | Help The Optimizer By Saving A Storage Variable’s Reference Instead Of Repeatedly Fetching It | 1 | - |
| [GAS&#x2011;13](#GAS&#x2011;13) | Superfluous event fields | 1 | - |
| [GAS&#x2011;14](#GAS&#x2011;14) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 3 | - |
| [GAS&#x2011;15](#GAS&#x2011;15) | Use assembly to check for `address(0)` | 1 | - |
| [GAS&#x2011;16](#GAS&#x2011;16) | Use of Custom Errors Instead of String | 3 | - |
| [GAS&#x2011;17](#GAS&#x2011;17) | Use solidity version 0.8.17 to gain some gas boost | 10 | 880 |
| [GAS&#x2011;18](#GAS&#x2011;18) | Using `storage` instead of `memory` saves gas | 1 | 350 |

Total: 109 contexts over 18 issues

## Gas Optimizations






### <a href="#Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Comparing to constant boolean

There is no need to verify that `== true` or `== false` when the variable checked upon is a boolean as well.

#### <ins>Proof Of Concept</ins>


```solidity
73: if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73

```solidity
221: if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221




#### <ins>Recommended Mitigation Steps</ins>

Instead simply check for `variable` or `!variable`



### <a href="#Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> State variables only set in the `constructor` should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmacces (100 gas) with a PUSH32 (3 gas).

#### <ins>Proof Of Concept</ins>

```solidity
43: questId = questId_
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L43




### <a href="#Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Setting the `constructor` to `payable`

Saves ~13 gas per instance

#### <ins>Proof Of Concept</ins>

```solidity
13: constructor(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountInWeiOrTokenId_,
        string memory questId_,
        address receiptContractAddress_
    )
    Quest(
        rewardTokenAddress_,
        endTime_,
        startTime_,
        totalParticipants_,
        rewardAmountInWeiOrTokenId_,
        questId_,
        receiptContractAddress_
    )
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L13

```solidity
17: constructor(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountInWeiOrTokenId_,
        string memory questId_,
        address receiptContractAddress_,
        uint256 questFee_,
        address protocolFeeRecipient_
    )
        Quest(
            rewardTokenAddress_,
            endTime_,
            startTime_,
            totalParticipants_,
            rewardAmountInWeiOrTokenId_,
            questId_,
            receiptContractAddress_
        )
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L17

```solidity
26: constructor(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountInWeiOrTokenId_,
        string memory questId_,
        address receiptContractAddress_
    )
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L26

```solidity
35: constructor() initializer
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L35

```solidity
39: constructor()
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L39

```solidity
28: constructor()
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L28





### <a href="#Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Empty Blocks Should Be Removed Or Emit Something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})

#### <ins>Proof Of Concept</ins>

```solidity
150: function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150






### <a href="#Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

```solidity
103: uint256 redeemableTokenCount = 0;

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L103

```solidity
115: uint foundTokens = 0;
126: uint filterTokensIndexTracker = 0;

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L115

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L126








### <a href="#Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry's script.sol and solmate's `LibRlp.sol` contracts can help achieve this.

References: 
https://book.getfoundry.sh/reference/forge-std/compute-create-address 

https://twitter.com/transmissions11/status/1518507047943245824

#### <ins>Proof Of Concept</ins>

```solidity
34: if (IERC1155(rewardToken).balanceOf(address(this), rewardAmountInWeiOrTokenId) < totalParticipants)

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L34

```solidity
42: IERC1155(rewardToken).safeTransferFrom(address(this), msg.sender, rewardAmountInWeiOrTokenId, amount_, '0x00');

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L42

```solidity
57: address(this),
60: IERC1155(rewardToken).balanceOf(address(this), rewardAmountInWeiOrTokenId),

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L57

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L60



```solidity
59: if (IERC20(rewardToken).balanceOf(address(this)) < maxTotalRewards() + maxProtocolReward())

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L59

```solidity
85: uint256 nonClaimableTokens = IERC20(rewardToken).balanceOf(address(this)) - protocolFee() - unclaimedTokens;

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L85



#### <ins>Recommended Mitigation Steps</ins>

Use hardcoded `address`



### <a href="#Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> `internal` functions only called once can be inlined to save gas

#### <ins>Proof Of Concept</ins>

```solidity
41: function _transferRewards

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L41

```solidity
48: function _calculateRewards

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L48

```solidity
66: function _transferRewards

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L66

```solidity
74: function _calculateRewards

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L74

```solidity
122: function _calculateRewards

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L122

```solidity
129: function _transferRewards

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L129

```solidity
152: function grantDefaultAdminAndCreateQuestRole

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152

```solidity
139: function _burn

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L139

```solidity
148: function _beforeTokenTransfer

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L148






### <a href="#Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.

#### <ins>Proof Of Concept</ins>


```solidity
20: mapping(address => bool) addressMinted;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L20

```solidity
30: mapping(address => bool) public rewardAllowlist;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L30






### <a href="#Summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

All in-scope contracts

#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.



### <a href="#Summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

#### <ins>Proof Of Concept</ins>


```solidity
115: redeemedTokens += redeemableTokenCount;

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115





### <a href="#Summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### <ins>Proof Of Concept</ins>


```solidity
function start() public override {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L33

```solidity
function withdrawRemainingTokens(address to_) public override onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54

```solidity
function maxTotalRewards() public view returns (uint256) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L45

```solidity
function maxProtocolReward() public view returns (uint256) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L52

```solidity
function start() public override {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L58

```solidity
function withdrawRemainingTokens(address to_) public override onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81

```solidity
function receiptRedeemers() public view returns (uint256) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L91

```solidity
function protocolFee() public view returns (uint256) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L96

```solidity
function withdrawFee() public onlyAdminWithdrawAfterEnd {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L102

```solidity
function start() public virtual onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50

```solidity
function pause() public onlyOwner onlyStarted {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57

```solidity
function unPause() public onlyOwner onlyStarted {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L63

```solidity
function claim() public virtual onlyQuestActive {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96

```solidity
function isClaimed(uint256 tokenId_) public view returns (bool) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L135

```solidity
function getRewardAmount() public view returns (uint256) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L140

```solidity
function getRewardToken() public view returns (address) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L145

```solidity
function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150

```solidity
function initialize(
        address claimSignerAddress_,
        address rabbitholeReceiptContract_,
        address protocolFeeRecipient_
    ) public initializer {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37

```solidity
function createQuest(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountOrTokenId_,
        string memory contractType_,
        string memory questId_
    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61

```solidity
function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L142

```solidity
function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159

```solidity
function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L165

```solidity
function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L172

```solidity
function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L179

```solidity
function setQuestFee(uint256 questFee_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L186

```solidity
function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L210

```solidity
function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219

```solidity
function initialize(
        address receiptRenderer_,
        address royaltyRecipient_,
        address minterAddress_,
        uint royaltyFee_
    ) public initializer {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L43

```solidity
function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L65

```solidity
function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71

```solidity
function setQuestFactory(address questFactory_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L77

```solidity
function setMinterAddress(address minterAddress_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L83

```solidity
function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90

```solidity
function mint(address to_, string memory questId_) public onlyMinter {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L98

```solidity
function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L109

```solidity
function tokenURI(
        uint tokenId_
    ) public view virtual override(ERC721Upgradeable, ERC721URIStorageUpgradeable) returns (string memory) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L158

```solidity
function supportsInterface(
        bytes4 interfaceId_
    ) public view virtual override(ERC721Upgradeable, ERC721EnumerableUpgradeable, IERC165Upgradeable) returns (bool) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L190

```solidity
function initialize(
        address ticketRenderer_,
        address royaltyRecipient_,
        address minterAddress_,
        uint royaltyFee_
    ) public initializer {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L32

```solidity
function setTicketRenderer(address ticketRenderer_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L54

```solidity
function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L60

```solidity
function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66

```solidity
function setMinterAddress(address minterAddress_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L73

```solidity
function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L83

```solidity
function mintBatch(
        address to_,
        uint256[] memory ids_,
        uint256[] memory amounts_,
        bytes memory data_
    ) public onlyMinter {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L92

```solidity
function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L102

```solidity
function supportsInterface(
        bytes4 interfaceId_
    ) public view virtual override(ERC1155Upgradeable, IERC165Upgradeable) returns (bool) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L119

```solidity
function generateTokenURI(
        uint tokenId_,
        string memory questId_,
        uint totalParticipants_,
        bool claimed_,
        uint rewardAmount_,
        address rewardAddress_
    ) public view virtual returns (string memory) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L21

```solidity
function generateDataURI(
        uint tokenId_,
        string memory questId_,
        uint totalParticipants_,
        bool claimed_,
        uint rewardAmount_,
        address rewardAddress_
    ) public view virtual returns (bytes memory) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L40

```solidity
function generateAttribute(string memory key, string memory value) public pure returns (string memory) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L82

```solidity
function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L100

```solidity
function generateTokenURI(
        uint tokenId_
    ) public pure returns (string memory) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L16

```solidity
function generateSVG(uint tokenId_) public pure returns (string memory) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L36









### <a href="#Summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Help The Optimizer By Saving A Storage Variable's Reference Instead Of Repeatedly Fetching It

To help the optimizer, declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array.
The effect can be quite significant.
As an example, instead of repeatedly calling someMap[someIndex], save its reference like this: SomeStruct storage someStruct = someMap[someIndex] and use it.

#### <ins>Proof Of Concept</ins>


```solidity
119: if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L119






### <a href="#Summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> Superfluous event fields

`block.number` and `block.timestamp` are added to the event information by default, so adding them manually will waste additional gas.

#### <ins>Proof Of Concept</ins>


```solidity
16: event QuestCreated(address indexed creator, address indexed contractAddress, string indexed questId, string contractType, address rewardTokenAddress, uint256 endTime, uint256 startTime, uint256 totalParticipants, uint256 rewardAmountOrTokenId);
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L16





### <a href="#Summary">[GAS&#x2011;14]</a><a name="GAS&#x2011;14"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
99: uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L99

```solidity
114: uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L114

```solidity
125: uint[] memory filteredTokens = new uint[](foundTokens);
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L125




### <a href="#Summary">[GAS&#x2011;15]</a><a name="GAS&#x2011;15"> Use assembly to check for `address(0)`

Save 6 gas per instance if using assembly to check for address(0)

e.g.
```
assembly {
	if iszero(_addr) {
		mstore(0x00, "AddressZero")
		revert(0x00, 0x20)
	}
}
```

#### <ins>Proof Of Concept</ins>


```solidity
function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L165






### <a href="#Summary">[GAS&#x2011;16]</a><a name="GAS&#x2011;16"> Use of Custom Errors Instead of String

To save some gas the use of custom errors leads to cheaper deploy time cost and run time cost. The run time cost is only relevant when the revert condition is met.

#### <ins>Proof Of Concept</ins>


```solidity
File: ./Projects/rabbithold2023-01/2023-01-rabbithole/quest-protocol/contracts/RabbitHoleReceipt.sol
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L182



#### <ins>Recommended Mitigation Steps</ins>

Use Custom Errors instead of strings.



### <a href="#Summary">[GAS&#x2011;17]</a><a name="GAS&#x2011;17"> Use solidity version 0.8.17 to gain some gas boost

Upgrade to the latest solidity version 0.8.17 to get additional gas savings.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L2

```solidity
pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L2





### <a href="#Summary">[GAS&#x2011;18]</a><a name="GAS&#x2011;18"> Using `storage` instead of `memory` saves gas

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another `memory` array/struct

#### <ins>Proof Of Concept</ins>


```solidity
164: string memory questId = questIdForTokenId[tokenId_];

```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L164





### <a href="#Summary">[GAS&#x2011;27]</a><a name="GAS&#x2011;27"> Use `uint256(1)`/`uint256(2)` instead for `true` and `false` boolean states

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from `true` to `false` can cost up to ~20000 gas rather than `uint256(2)` to `uint256(1)` that would cost significantly less.

#### <ins>Proof Of Concept</ins>


```solidity
24: mapping(uint256 => bool) private claimedList;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L24

```solidity
51: isPaused = false;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L51

```solidity
64: isPaused = false;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L64

```solidity
52: hasStarted = true;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L52

```solidity
58: isPaused = true;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L58

```solidity
71: claimedList[tokenIds_[i]] = true;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L71

```solidity
20: mapping(address => bool) addressMinted;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L20

```solidity
30: mapping(address => bool) public rewardAllowlist;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L30

```solidity
225: quests[questId_].addressMinted[msg.sender] = true;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L225






### <a href="#Summary">[GAS&#x2011;28]</a><a name="GAS&#x2011;28"> Using Bools For Storage Incurs Overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.

#### <ins>Proof Of Concept</ins>


```solidity
19: bool public hasStarted;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L19

```solidity
20: bool public isPaused;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L20

```solidity
24: mapping(uint256 => bool) private claimedList;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L24

```solidity
20: mapping(address => bool) addressMinted;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L20

```solidity
30: mapping(address => bool) public rewardAllowlist;
```

https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L30





