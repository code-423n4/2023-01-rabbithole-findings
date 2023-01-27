
### [L01] Missing checks for `address(0x0)` when assigning values to `address` state variables


#### Findings:
```
quest-protocol/contracts/Erc20Quest.sol::38 => questFee = questFee_;
quest-protocol/contracts/Erc20Quest.sol::39 => protocolFeeRecipient = protocolFeeRecipient_;
quest-protocol/contracts/Quest.sol::38 => endTime = endTime_;
quest-protocol/contracts/Quest.sol::39 => startTime = startTime_;
quest-protocol/contracts/Quest.sol::40 => rewardToken = rewardTokenAddress_;
quest-protocol/contracts/Quest.sol::41 => totalParticipants = totalParticipants_;
quest-protocol/contracts/Quest.sol::42 => rewardAmountInWeiOrTokenId = rewardAmountInWeiOrTokenId_;
quest-protocol/contracts/Quest.sol::43 => questId = questId_;
quest-protocol/contracts/Quest.sol::44 => rabbitHoleReceiptContract = RabbitHoleReceipt(receiptContractAddress_);
quest-protocol/contracts/QuestFactory.sol::45 => claimSignerAddress = claimSignerAddress_;
quest-protocol/contracts/QuestFactory.sol::160 => claimSignerAddress = claimSignerAddress_;
quest-protocol/contracts/QuestFactory.sol::167 => protocolFeeRecipient = protocolFeeRecipient_;
quest-protocol/contracts/QuestFactory.sol::188 => questFee = questFee_;
quest-protocol/contracts/RabbitHoleReceipt.sol::52 => royaltyRecipient = royaltyRecipient_;
quest-protocol/contracts/RabbitHoleReceipt.sol::53 => minterAddress = minterAddress_;
quest-protocol/contracts/RabbitHoleReceipt.sol::54 => royaltyFee = royaltyFee_;
quest-protocol/contracts/RabbitHoleReceipt.sol::72 => royaltyRecipient = royaltyRecipient_;
quest-protocol/contracts/RabbitHoleReceipt.sol::84 => minterAddress = minterAddress_;
quest-protocol/contracts/RabbitHoleReceipt.sol::91 => royaltyFee = royaltyFee_;
quest-protocol/contracts/RabbitHoleTickets.sol::41 => royaltyRecipient = royaltyRecipient_;
quest-protocol/contracts/RabbitHoleTickets.sol::42 => minterAddress = minterAddress_;
quest-protocol/contracts/RabbitHoleTickets.sol::43 => royaltyFee = royaltyFee_;
quest-protocol/contracts/RabbitHoleTickets.sol::61 => royaltyRecipient = royaltyRecipient_;
quest-protocol/contracts/RabbitHoleTickets.sol::67 => royaltyFee = royaltyFee_;
quest-protocol/contracts/RabbitHoleTickets.sol::74 => minterAddress = minterAddress_;
```




### [L02] `initialize` functions can be front-run

#### Impact
See [this](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/40) finding from a prior badger-dao contest for details
#### Findings:
```
quest-protocol/contracts/QuestFactory.sol::35 => constructor() initializer {}
quest-protocol/contracts/QuestFactory.sol::37 => function initialize(
quest-protocol/contracts/QuestFactory.sol::41 => ) public initializer {
quest-protocol/contracts/RabbitHoleReceipt.sol::43 => function initialize(
quest-protocol/contracts/RabbitHoleReceipt.sol::48 => ) public initializer {
quest-protocol/contracts/RabbitHoleTickets.sol::32 => function initialize(
quest-protocol/contracts/RabbitHoleTickets.sol::37 => ) public initializer {
```




### [L03] Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions

#### Impact
See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.
#### Findings:
```
quest-protocol/contracts/QuestFactory.sol::16 => contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory {
quest-protocol/contracts/RabbitHoleReceipt.sol::15-22 => contract RabbitHoleReceipt is
    Initializable,
    ERC721Upgradeable,
    ERC721EnumerableUpgradeable,
    ERC721URIStorageUpgradeable,
    OwnableUpgradeable,
    IERC2981Upgradeable
{
quest-protocol/contracts/RabbitHoleTickets.sol::11-17 => contract RabbitHoleTickets is
    Initializable,
    ERC1155Upgradeable,
    OwnableUpgradeable,
    ERC1155BurnableUpgradeable,
    IERC2981Upgradeable
{
```

### [L04] Unspecific Compiler Version Pragma

#### Impact
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.
#### Findings:
```
quest-protocol/contracts/Erc1155Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/Erc20Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/QuestFactory.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/RabbitHoleReceipt.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/RabbitHoleTickets.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/ReceiptRenderer.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/TicketRenderer.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/interfaces/IQuest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/interfaces/IQuestFactory.sol::2 => pragma solidity ^0.8.15;
```



### Non-Critical Issues

### [N01] Adding a return statement when the function defines a named return variable, is redundant



#### Findings:
```
quest-protocol/contracts/Erc1155Quest.sol::49 => return redeemableTokenCount_;
quest-protocol/contracts/Quest.sol::141 => return rewardAmountInWeiOrTokenId;
quest-protocol/contracts/Quest.sol::146 => return rewardToken;
quest-protocol/contracts/RabbitHoleReceipt.sol::134 => return filteredTokens;
quest-protocol/contracts/ReceiptRenderer.sol::76 => return dataURI;
```


### [N02] constants should be defined rather than using magic numbers


#### Findings:
```
quest-protocol/contracts/Erc20Quest.sol::53 => return (maxTotalRewards() * questFee) / 10_000;
quest-protocol/contracts/Erc20Quest.sol::97 => return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
quest-protocol/contracts/RabbitHoleReceipt.sol::184 => uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
quest-protocol/contracts/RabbitHoleTickets.sol::113 => uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```




### [N03] Event is missing `indexed` fields

#### Impact
Each event should use three indexed fields if there are three or more fields
#### Findings:
```
quest-protocol/contracts/interfaces/IQuest.sol::8 => event Claimed(address account, uint256 amount);
```





### [N04] Open TODOs

#### Impact
Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment
#### Findings:
```
quest-protocol/contracts/interfaces/IQuest.sol::4 => // TODO clean this whole thing up
```



### [N05] Unused file


#### Findings:
```
quest-protocol/contracts/RabbitHoleReceipt.sol::1 => // SPDX-License-Identifier: MIT
quest-protocol/contracts/RabbitHoleTickets.sol::1 => // SPDX-License-Identifier: MIT
quest-protocol/contracts/ReceiptRenderer.sol::1 => // SPDX-License-Identifier: MIT
quest-protocol/contracts/TicketRenderer.sol::1 => // SPDX-License-Identifier: MIT
```




### [N06] `public` functions not called by the contract should be declared `external` instead

#### Impact
Contracts are allowed to override their parents’ functions and change the visibility from public to external .
#### Findings:
```
quest-protocol/contracts/Erc1155Quest.sol::33 => function start() public override {
quest-protocol/contracts/Erc1155Quest.sol::54 => function withdrawRemainingTokens(address to_) public override onlyOwner {
quest-protocol/contracts/Erc20Quest.sol::45 => function maxTotalRewards() public view returns (uint256) {
quest-protocol/contracts/Erc20Quest.sol::52 => function maxProtocolReward() public view returns (uint256) {
quest-protocol/contracts/Erc20Quest.sol::58 => function start() public override {
quest-protocol/contracts/Erc20Quest.sol::81 => function withdrawRemainingTokens(address to_) public override onlyOwner {
quest-protocol/contracts/Erc20Quest.sol::91 => function receiptRedeemers() public view returns (uint256) {
quest-protocol/contracts/Erc20Quest.sol::96 => function protocolFee() public view returns (uint256) {
quest-protocol/contracts/Erc20Quest.sol::102 => function withdrawFee() public onlyAdminWithdrawAfterEnd {
quest-protocol/contracts/Quest.sol::50 => function start() public virtual onlyOwner {
quest-protocol/contracts/Quest.sol::57 => function pause() public onlyOwner onlyStarted {
quest-protocol/contracts/Quest.sol::63 => function unPause() public onlyOwner onlyStarted {
quest-protocol/contracts/Quest.sol::96 => function claim() public virtual onlyQuestActive {
quest-protocol/contracts/Quest.sol::135 => function isClaimed(uint256 tokenId_) public view returns (bool) {
quest-protocol/contracts/Quest.sol::140 => function getRewardAmount() public view returns (uint256) {
quest-protocol/contracts/Quest.sol::145 => function getRewardToken() public view returns (address) {
quest-protocol/contracts/Quest.sol::150 => function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
quest-protocol/contracts/QuestFactory.sol::69 => ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
quest-protocol/contracts/QuestFactory.sol::142 => function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::159 => function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::165 => function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::172 => function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::179 => function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::186 => function setQuestFee(uint256 questFee_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::210 => function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) {
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
quest-protocol/contracts/RabbitHoleTickets.sol::83 => function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {
quest-protocol/contracts/RabbitHoleTickets.sol::102 => function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {
quest-protocol/contracts/RabbitHoleTickets.sol::121 => ) public view virtual override(ERC1155Upgradeable, IERC165Upgradeable) returns (bool) {
quest-protocol/contracts/ReceiptRenderer.sol::28 => ) public view virtual returns (string memory) {
quest-protocol/contracts/ReceiptRenderer.sol::47 => ) public view virtual returns (bytes memory) {
quest-protocol/contracts/ReceiptRenderer.sol::82 => function generateAttribute(string memory key, string memory value) public pure returns (string memory) {
quest-protocol/contracts/ReceiptRenderer.sol::100 => function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {
quest-protocol/contracts/TicketRenderer.sol::18 => ) public pure returns (string memory) {
quest-protocol/contracts/TicketRenderer.sol::36 => function generateSVG(uint tokenId_) public pure returns (string memory) {
```



### [N07] Constant redefined elsewhere

#### Impact
Consider defining in only one contract so that values cannot become out of sync when only one location is updated
#### Findings:
```
quest-protocol/contracts/QuestFactory.sol::17 => bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');

```




### [N08] NC-library/interface files should use fixed compiler versions, not floating ones


#### Findings:
```
quest-protocol/contracts/Erc1155Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/Erc20Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/QuestFactory.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/RabbitHoleReceipt.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/RabbitHoleTickets.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/ReceiptRenderer.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/TicketRenderer.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/interfaces/IQuest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/interfaces/IQuestFactory.sol::2 => pragma solidity ^0.8.15;
```






