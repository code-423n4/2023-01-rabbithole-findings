# mapping value should be cached rather than re-indexed from storage
in [QuestFactory.sol#L28](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L28),  within one function __quests__ is used  multiple-times, in this case, __quests__ should be cached to save gas.
we can use the follow code as poc


    // SPDX-License-Identifier: GPL-3.0
 
    pragma solidity ^0.8.15;
 
    contract StorageA {
        struct A{
            uint a;
            uint b;
            uint c;
            uint d;
        }
 
        mapping (string => A) m;
 
        constructor() {
            A memory a;
            a.a = 1;
            a.b = 1;
            a.c = 1;
            a.d = 1;
            m["a"] = a;
        }
 
        function show(string calldata key) public view returns (uint, uint, uint, uint) {
            return (m[key].a, m[key].b, m[key].c, m[key].d);
        }
 
        function setV(string calldata key, uint v) public {
            m[key].a = v;
            m[key].b = v;
            m[key].c = v;
            m[key].d = v;
        }
    }
 
    contract StorageB {
        struct A{
            uint a;
            uint b;
            uint c;
            uint d;
        }
 
        mapping (string => A) m;
 
        constructor() {
            A memory a;
            a.a = 1;
            a.b = 1;
            a.c = 1;
            a.d = 1;
            m["a"] = a;
        }
 
        function show(string calldata key) public view returns (uint, uint, uint, uint) {
            A memory a = m[key];
            return (a.a, a.b, a.c, a.d);
        }
 
        function setV(string calldata key, uint v) public {
            A storage a = m[key];
 
            a.a = v;
            a.b = v;
            a.c = v;
            a.d = v;
        }
    }
 
 
    contract StorageC {
        struct A{
            uint a;
            uint b;
            uint c;
            uint d;
        }
    
        mapping (string => A) m;
    
        constructor() {
            A memory a;
            a.a = 1;
            a.b = 1;
            a.c = 1;
            a.d = 1;
            m["a"] = a;
        }
    
        function show(string calldata key) public view returns (uint, uint, uint, uint) {
            A storage a = m[key];
            return (a.a, a.b, a.c, a.d);
        }
    
        function setV(string calldata key, uint v) public {
            A storage a = m[key];
    
            a.a = v;
            a.b = v;
            a.c = v;
            a.d = v;
        }
    }

as we can see, contract StorageC can save more gas


# timestampForTokenId is never read or used, so remove the related code
timestampForTokenId defined in [RabbitHoleReceipt.sol#L34](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L34) is only been set in [RabbitHoleReceipt.sol#L102](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L102), but never been read, so maybe it's useless.

# stack variable used as a cheaper cache for a state variable is only used once
If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.
[RabbitHoleReceipt.sol#L164](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L164)

# Internal functions only called once can be inlined to save gas
[QuestFactory.sol#L152](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152)

# <x> += <y> costs more gas than <x> = <x> + <y> for state variables
[Quest.sol#L115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115)

# Don’t compare boolean expressions to boolean literals
    if (<x> == true) => if (<x>),
    if (<x> == false) => if (!<x>)
[QuestFactory.sol#L73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73) and [QuestFactory.sol#L221](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221) and [Quest.sol#L135](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L135)


# Functions guaranteed to revert when called by normal users can be marked payable
  If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

[RabbitHoleTickets.sol#L83](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L83) and [RabbitHoleTickets.sol#L97](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L97) and [RabbitHoleReceipt.sol#L98](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L98)

# Not using the named return variables when a function returns, wastes deployment gas 
in [RabbitHoleReceipt.sol#L181](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L181)

in [RabbitHoleTickets.sol#L112](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L112)

# Public functions not called by the contract should be declared external instead 
in [Erc1155Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol)

        @@ -33 +33 @@ contract Erc1155Quest is Quest, ERC1155Holder {
        -    function start() public override {
        +    function start() external public override {
        @@ -54 +54 @@ contract Erc1155Quest is Quest, ERC1155Holder {
        -    function withdrawRemainingTokens(address to_) public override onlyOwner {
        +    function withdrawRemainingTokens(address to_) external override onlyOwner {

in [Erc20Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol)

        @@ -58 +58 @@ contract Erc20Quest is Quest {
        -    function start() public override {
        +    function start() external override {
        @@ -81 +81 @@ contract Erc20Quest is Quest {
        -    function withdrawRemainingTokens(address to_) public override onlyOwner {
        +    function withdrawRemainingTokens(address to_) external override onlyOwner {
        @@ -102 +102 @@ contract Erc20Quest is Quest {
        -    function withdrawFee() public onlyAdminWithdrawAfterEnd {
        +    function withdrawFee() external onlyAdminWithdrawAfterEnd {

in [Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol)

        @@ -57 +57 @@ contract Quest is Ownable, IQuest {
        -    function pause() public onlyOwner onlyStarted {
        +    function pause() external onlyOwner onlyStarted {
        @@ -63 +63 @@ contract Quest is Ownable, IQuest {
        -    function unPause() public onlyOwner onlyStarted {
        +    function unPause() external onlyOwner onlyStarted {
        @@ -96 +96 @@ contract Quest is Ownable, IQuest {
        -    function claim() public virtual onlyQuestActive {
        +    function claim() external virtual onlyQuestActive {
        @@ -140 +140 @@ contract Quest is Ownable, IQuest {
        -    function getRewardAmount() public view returns (uint256) {
        +    function getRewardAmount() external view returns (uint256) {
        @@ -145 +145 @@ contract Quest is Ownable, IQuest {
        -    function getRewardToken() public view returns (address) {
        +    function getRewardToken() external view returns (address) {


in [QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)

    @@ -41 +41 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
    -    ) public initializer {
    +    ) external initializer {
    @@ -69 +69 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
    -    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
    +    ) external onlyRole(CREATE_QUEST_ROLE) returns (address) {
    @@ -142 +142 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
    -    function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
    +    function changeCreateQuestRole(address account_, bool canCreateQuest_) external onlyOwner {
    @@ -159 +159 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
    -    function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
    +    function setClaimSignerAddress(address claimSignerAddress_) external onlyOwner {
    @@ -172 +172 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
    -    function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
    +    function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) external onlyOwner {
    @@ -179 +179 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
    -    function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
    +    function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) external onlyOwner {
    @@ -219 +219 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
    -    function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
    +    function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) external {


in [RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol)

    @@ -48 +48 @@ contract RabbitHoleReceipt is
    -    ) public initializer {
    +    ) external initializer {
    @@ -65 +65 @@ contract RabbitHoleReceipt is
    -    function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
    +    function setReceiptRenderer(address receiptRenderer_) external onlyOwner {
    @@ -71 +71 @@ contract RabbitHoleReceipt is
    -    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
    +    function setRoyaltyRecipient(address royaltyRecipient_) external onlyOwner {
    @@ -77 +77 @@ contract RabbitHoleReceipt is
    -    function setQuestFactory(address questFactory_) public onlyOwner {
    +    function setQuestFactory(address questFactory_) external onlyOwner {
    @@ -83 +83 @@ contract RabbitHoleReceipt is
    -    function setMinterAddress(address minterAddress_) public onlyOwner {
    +    function setMinterAddress(address minterAddress_) external onlyOwner {
    @@ -90 +90 @@ contract RabbitHoleReceipt is
    -    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
    +    function setRoyaltyFee(uint256 royaltyFee_) external onlyOwner {
    @@ -98 +98 @@ contract RabbitHoleReceipt is
    -    function mint(address to_, string memory questId_) public onlyMinter {
    +    function mint(address to_, string memory questId_) external onlyMinter {
    @@ -112 +112 @@ contract RabbitHoleReceipt is
    -    ) public view returns (uint[] memory) {
    +    ) external view returns (uint[] memory) {


in [RabbitHoleTickets.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol)

        @@ -54 +54 @@ contract RabbitHoleTickets is
        -    function setTicketRenderer(address ticketRenderer_) public onlyOwner {
        +    function setTicketRenderer(address ticketRenderer_) external onlyOwner {
        @@ -60 +60 @@ contract RabbitHoleTickets is
        -    function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
        +    function setRoyaltyRecipient(address royaltyRecipient_) external  onlyOwner {
        @@ -66 +66 @@ contract RabbitHoleTickets is
        -    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        +    function setRoyaltyFee(uint256 royaltyFee_) external onlyOwner {
        @@ -73 +73 @@ contract RabbitHoleTickets is
        -    function setMinterAddress(address minterAddress_) public onlyOwner {
        +    function setMinterAddress(address minterAddress_) external onlyOwner {
        @@ -83 +83 @@ contract RabbitHoleTickets is
        -    function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {
        +    function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) external onlyMinter {
        @@ -97 +97 @@ contract RabbitHoleTickets is
        -    ) public onlyMinter {
        +    ) external onlyMinter {


in [ReceiptRenderer.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol)

        @@ -28 +28 @@ contract ReceiptRenderer {
        -    ) public view virtual returns (string memory) {
        +    ) external view virtual returns (string memory) {

in [TicketRenderer.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol)

        @@ -18 +18 @@ contract TicketRenderer {
        -    ) public pure returns (string memory) {
        +    ) external pure returns (string memory) {
