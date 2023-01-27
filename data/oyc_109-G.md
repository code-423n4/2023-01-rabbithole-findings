## [G-01] Don't Initialize Variables with Default Value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecesary gas.

```
quest-protocol/contracts/Quest.sol::70 => for (uint i = 0; i < tokenIds_.length; i++) {
quest-protocol/contracts/Quest.sol::103 => uint256 redeemableTokenCount = 0;
quest-protocol/contracts/Quest.sol::104 => for (uint i = 0; i < tokens.length; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::115 => uint foundTokens = 0;
quest-protocol/contracts/RabbitHoleReceipt.sol::117 => for (uint i = 0; i < msgSenderBalance; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::126 => uint filterTokensIndexTracker = 0;
quest-protocol/contracts/RabbitHoleReceipt.sol::128 => for (uint i = 0; i < msgSenderBalance; i++) {
```

## [G-02] Cache Array Length Outside of Loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

```
quest-protocol/contracts/Quest.sol::70 => for (uint i = 0; i < tokenIds_.length; i++) {
quest-protocol/contracts/Quest.sol::104 => for (uint i = 0; i < tokens.length; i++) {
```

## [G-03] Use Shift Right/Left instead of Division/Multiplication if possible

A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

```
quest-protocol/contracts/ReceiptRenderer.sol::102 => '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
quest-protocol/contracts/TicketRenderer.sol::38 => '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
```

## [G-04] Use calldata instead of memory

Use calldata instead of memory for function parameters saves gas if the function argument is only read.

```
quest-protocol/contracts/QuestFactory.sol::193 => function getNumberMinted(string memory questId_) external view returns (uint) {
quest-protocol/contracts/QuestFactory.sol::199 => function questInfo(string memory questId_) external view returns (address, uint, uint) {
quest-protocol/contracts/QuestFactory.sol::210 => function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) {
quest-protocol/contracts/ReceiptRenderer.sol::82 => function generateAttribute(string memory key, string memory value) public pure returns (string memory) {
quest-protocol/contracts/ReceiptRenderer.sol::100 => function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {
quest-protocol/contracts/interfaces/IQuestFactory.sol::19 => function questInfo(string memory questId_) external view returns (address, uint, uint);
```

## [G-05] Using private rather than public for constants, saves gas

If needed, the value can be read from the verified contract source code. Savings are due to the compiler not having to create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table

```
quest-protocol/contracts/Erc20Quest.sol::13 => uint256 public immutable questFee;
quest-protocol/contracts/Erc20Quest.sol::14 => address public immutable protocolFeeRecipient;
quest-protocol/contracts/Erc20Quest.sol::15 => QuestFactory public immutable questFactoryContract;
quest-protocol/contracts/Quest.sol::13 => RabbitHoleReceipt public immutable rabbitHoleReceiptContract;
quest-protocol/contracts/Quest.sol::14 => address public immutable rewardToken;
quest-protocol/contracts/Quest.sol::15 => uint256 public immutable endTime;
quest-protocol/contracts/Quest.sol::16 => uint256 public immutable startTime;
quest-protocol/contracts/Quest.sol::17 => uint256 public immutable totalParticipants;
quest-protocol/contracts/Quest.sol::18 => uint256 public immutable rewardAmountInWeiOrTokenId;
```

## [G-06] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```
quest-protocol/contracts/Erc1155Quest.sol::54 => function withdrawRemainingTokens(address to_) public override onlyOwner {
quest-protocol/contracts/Erc20Quest.sol::81 => function withdrawRemainingTokens(address to_) public override onlyOwner {
quest-protocol/contracts/Erc20Quest.sol::102 => function withdrawFee() public onlyAdminWithdrawAfterEnd {
quest-protocol/contracts/Quest.sol::50 => function start() public virtual onlyOwner {
quest-protocol/contracts/Quest.sol::57 => function pause() public onlyOwner onlyStarted {
quest-protocol/contracts/Quest.sol::63 => function unPause() public onlyOwner onlyStarted {
quest-protocol/contracts/Quest.sol::96 => function claim() public virtual onlyQuestActive {
quest-protocol/contracts/Quest.sol::150 => function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
quest-protocol/contracts/QuestFactory.sol::142 => function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::159 => function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::165 => function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::172 => function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::179 => function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::186 => function setQuestFee(uint256 questFee_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::65 => function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::71 => function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::77 => function setQuestFactory(address questFactory_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::83 => function setMinterAddress(address minterAddress_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::90 => function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::98 => function mint(address to_, string memory questId_) public onlyMinter {
quest-protocol/contracts/RabbitHoleTickets.sol::54 => function setTicketRenderer(address ticketRenderer_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::60 => function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::66 => function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::73 => function setMinterAddress(address minterAddress_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::83 => function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {
```

## [G-07] Using bools for storage incurs overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.
Use uint256(1) and uint256(2) for true/false instead

```
quest-protocol/contracts/Quest.sol::19 => bool public hasStarted;
quest-protocol/contracts/Quest.sol::20 => bool public isPaused;
quest-protocol/contracts/Quest.sol::24 => mapping(uint256 => bool) private claimedList;
quest-protocol/contracts/QuestFactory.sol::30 => mapping(address => bool) public rewardAllowlist;
```

## [G-08] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, for example when used in for- and while-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop

```
quest-protocol/contracts/Quest.sol::70 => for (uint i = 0; i < tokenIds_.length; i++) {
quest-protocol/contracts/Quest.sol::104 => for (uint i = 0; i < tokens.length; i++) {
quest-protocol/contracts/Quest.sol::106 => redeemableTokenCount++;
quest-protocol/contracts/QuestFactory.sol::101 => ++questIdCount;
quest-protocol/contracts/QuestFactory.sol::132 => ++questIdCount;
quest-protocol/contracts/QuestFactory.sol::226 => quests[questId_].numberMinted++;
quest-protocol/contracts/RabbitHoleReceipt.sol::117 => for (uint i = 0; i < msgSenderBalance; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::121 => foundTokens++;
quest-protocol/contracts/RabbitHoleReceipt.sol::128 => for (uint i = 0; i < msgSenderBalance; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::131 => filterTokensIndexTracker++;
```

## [G-09] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

use <x> = <x> + <y> or <x> = <x> - <y> instead to save gas

```
quest-protocol/contracts/Quest.sol::115 => redeemedTokens += redeemableTokenCount;
```

## [G-10] Prefix increments cheaper than Postfix increments

++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)
Saves 5 gas PER LOOP

```
quest-protocol/contracts/Quest.sol::70 => for (uint i = 0; i < tokenIds_.length; i++) {
quest-protocol/contracts/Quest.sol::104 => for (uint i = 0; i < tokens.length; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::117 => for (uint i = 0; i < msgSenderBalance; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::128 => for (uint i = 0; i < msgSenderBalance; i++) {
```

## [G-11] Don't compare boolean expressions to boolean literals

The extran comparison wastes gas
if (<x> == true) => if (<x>), if (<x> == false) => if (!<x>)

```
quest-protocol/contracts/Quest.sol::136 => return claimedList[tokenId_] == true;
quest-protocol/contracts/QuestFactory.sol::221 => if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```

## [G-12] Public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.

```
quest-protocol/contracts/Erc20Quest.sol::102 => function withdrawFee() public onlyAdminWithdrawAfterEnd {
quest-protocol/contracts/Quest.sol::50 => function start() public virtual onlyOwner {
quest-protocol/contracts/Quest.sol::57 => function pause() public onlyOwner onlyStarted {
quest-protocol/contracts/Quest.sol::63 => function unPause() public onlyOwner onlyStarted {
quest-protocol/contracts/Quest.sol::96 => function claim() public virtual onlyQuestActive {
quest-protocol/contracts/Quest.sol::140 => function getRewardAmount() public view returns (uint256) {
quest-protocol/contracts/Quest.sol::145 => function getRewardToken() public view returns (address) {
quest-protocol/contracts/Quest.sol::150 => function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
quest-protocol/contracts/QuestFactory.sol::142 => function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::159 => function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::172 => function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::179 => function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::219 => function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
quest-protocol/contracts/RabbitHoleReceipt.sol::65 => function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::71 => function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::77 => function setQuestFactory(address questFactory_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::83 => function setMinterAddress(address minterAddress_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::90 => function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::98 => function mint(address to_, string memory questId_) public onlyMinter {
quest-protocol/contracts/RabbitHoleTickets.sol::54 => function setTicketRenderer(address ticketRenderer_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::60 => function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::66 => function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::73 => function setMinterAddress(address minterAddress_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::102 => function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {
```

## [G-13] Not using the named return variables when a function returns, wastes deployment gas

It is not necessary to have both a named return and a return statement.

```
quest-protocol/contracts/RabbitHoleReceipt.sol::181 => ) external view override returns (address receiver, uint256 royaltyAmount) {
quest-protocol/contracts/RabbitHoleTickets.sol::112 => ) external view override returns (address receiver, uint256 royaltyAmount) {
```

## [G-14] Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

```
quest-protocol/contracts/QuestFactory.sol::20 => mapping(address => bool) addressMinted;
quest-protocol/contracts/QuestFactory.sol::30 => mapping(address => bool) public rewardAllowlist;
```

## [G-15] internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```
quest-protocol/contracts/Quest.sol::122 => function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {
quest-protocol/contracts/Quest.sol::129 => function _transferRewards(uint256 amount_) internal virtual {
quest-protocol/contracts/QuestFactory.sol::152 => function grantDefaultAdminAndCreateQuestRole(address account_) internal {
```