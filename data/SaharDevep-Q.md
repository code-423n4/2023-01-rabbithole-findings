# Audit Report

## Summery
[L01] Unspecific Compiler Version Pragma
[L02] Use _transferOwnership internal function instead of transferOwnership function
[L03] The modifier does not revert
[L04] tokenUri() revert for tokens that don't implement IERC20metadata
[N01] Lines are too long
[N02] Useless getter function 
[N03] Explicitly use uint256 instead of uint
[N04] Not using the named variables anywhere in the function is confusing
[N05] Expressions for constant values such as a call to keccak256(), should use immutabel rather than constant
[N06] Unused import
[N07] Constants should be defined rather than magic numbers
[N08] Override function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings


## Issues found

### [L01] Unspecific Compiler Version Pragma
While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different evm compilation that is ultimately deployed on the blockchain.

#### Impact
Issue Information: (https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l003---unspecific-compiler-version-pragma)

#### Findings:
```
quest-protocol\contracts\Erc1155Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\Erc20Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\QuestFactory.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\RabbitHoleReceipt.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\RabbitHoleTickets.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\ReceiptRenderer.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\TicketRenderer.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\interfaces\IQuest.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\interfaces\IQuestFactory.sol::2 => pragma solidity ^0.8.15;
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)


### [L02] Use _transferOwnership internal function instead of transferOwnership function

#### Findings:
```
quest-protocol\contracts\QuestFactory.sol::100 => newQuest.transferOwnership(msg.sender);
quest-protocol\contracts\QuestFactory.sol::131 => newQuest.transferOwnership(msg.sender);
```

#### Tools used
Manual

### [L03] The modifier does not revert

#### Findings:
```
quest-protocol\contracts\RabbitHoleTickets.sol::47 => modifier onlyMinter() {
quest-protocol\contracts\RabbitHoleReceipt.sol::58 => modifier onlyMinter() {
```

#### Tools used
Manual

### [L04] tokenUri() revert for tokens that don't implement IERC20metadata
Use safeDecimals() instead.

#### Findings:
```
quest-protocol\contracts\RabbitHoleReceipt.sol::158 => function tokenURI(
```

#### Tools used
Manual

### [N01] Lines are too long
For better readabality, consider changing the line lengths to lower than 120 chars.

#### Findings:
```
quest-protocol\contracts\RabbitHoleReceipt.sol::172 => return ReceiptRendererContract.generateTokenURI(tokenId_, questId, totalParticipants, claimed, rewardAmount, rewardAddress);
```

#### Tools used
Manual

### [N02] Useless getter function 

#### Fidings:
```
quest-protocol\contracts\Quest.sol::145 => function getRewardToken() public view returns (address) {
quest-protocol\contracts\Quest.sol::140 => function getRewardAmount() public view returns (uint256) {
```

#### Tools used
Manual

### [N03] Explicitly use uint256 instead of uint

#### Fidings:
```
quest-protocol\contracts\Erc20Quest.sol::84 => uint unclaimedTokens = (receiptRedeemers() - redeemedTokens) * rewardAmountInWeiOrTokenId;
quest-protocol\contracts\Quest.sol::70 => for (uint i = 0; i < tokenIds_.length; i++) {
quest-protocol\contracts\Quest.sol::99 => uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);
quest-protocol\contracts\Quest.sol::104 => for (uint i = 0; i < tokens.length; i++) {
quest-protocol\contracts\QuestFactory.sol::22 => uint totalParticipants;
quest-protocol\contracts\QuestFactory.sol::23 => uint numberMinted;
quest-protocol\contracts\QuestFactory.sol::31 => uint public questFee;
quest-protocol\contracts\QuestFactory.sol::32 => uint public questIdCount;
quest-protocol\contracts\QuestFactory.sol::193 => function getNumberMinted(string memory questId_) external view returns (uint) {
quest-protocol\contracts\QuestFactory.sol::199 => function questInfo(string memory questId_) external view returns (address, uint, uint) {
quest-protocol\contracts\RabbitHoleReceipt.sol::30 => mapping(uint => string) public questIdForTokenId;
quest-protocol\contracts\RabbitHoleReceipt.sol::33 => uint public royaltyFee;
quest-protocol\contracts\RabbitHoleReceipt.sol::34 => mapping(uint => uint) public timestampForTokenId;
quest-protocol\contracts\RabbitHoleReceipt.sol::47 => uint royaltyFee_
quest-protocol\contracts\RabbitHoleReceipt.sol::100 => uint newTokenID = _tokenIds.current();
quest-protocol\contracts\RabbitHoleReceipt.sol::112 => ) public view returns (uint[] memory) {
quest-protocol\contracts\RabbitHoleReceipt.sol::113 => uint msgSenderBalance = balanceOf(claimingAddress_);
quest-protocol\contracts\RabbitHoleReceipt.sol::114 => uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
quest-protocol\contracts\RabbitHoleReceipt.sol::115 => uint foundTokens = 0;
quest-protocol\contracts\RabbitHoleReceipt.sol::117 => for (uint i = 0; i < msgSenderBalance; i++) {
quest-protocol\contracts\RabbitHoleReceipt.sol::118 => uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
quest-protocol\contracts\RabbitHoleReceipt.sol::125 => uint[] memory filteredTokens = new uint[](foundTokens);
quest-protocol\contracts\RabbitHoleReceipt.sol::126 => uint filterTokensIndexTracker = 0;
quest-protocol\contracts\RabbitHoleReceipt.sol::128 => for (uint i = 0; i < msgSenderBalance; i++) { 
quest-protocol\contracts\RabbitHoleReceipt.sol::159 => uint tokenId_
quest-protocol\contracts\RabbitHoleReceipt.sol::165 => (address questAddress, uint totalParticipants, ) = QuestFactoryContract.questInfo(questId);
quest-protocol\contracts\RabbitHoleReceipt.sol::169 => uint rewardAmount = questContract.getRewardAmount();
quest-protocol\contracts\RabbitHoleTickets.sol::24 => uint public royaltyFee;
quest-protocol\contracts\RabbitHoleTickets.sol::36 => uint royaltyFee_
quest-protocol\contracts\RabbitHoleTickets.sol::102 => function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {
quest-protocol\contracts\ReceiptRenderer.sol::22 => uint tokenId_,
quest-protocol\contracts\ReceiptRenderer.sol::24 => uint totalParticipants_,
quest-protocol\contracts\ReceiptRenderer.sol::26 => uint rewardAmount_,
quest-protocol\contracts\ReceiptRenderer.sol::41 => uint tokenId_,
quest-protocol\contracts\ReceiptRenderer.sol::43 => uint totalParticipants_,
quest-protocol\contracts\ReceiptRenderer.sol::45 => uint rewardAmount_,
quest-protocol\contracts\ReceiptRenderer.sol::100 => function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {
quest-protocol\contracts\TicketRenderer.sol::17 => uint tokenId_
quest-protocol\contracts\TicketRenderer.sol::36 => function generateSVG(uint tokenId_) public pure returns (string memory) {
quest-protocol\contracts\interfaces\IQuestFactory.sol::19 => function questInfo(string memory questId_) external view returns (address, uint, uint);
```

#### Tools used
Manual

### [N04] Not using the named variables anywhere in the function is confusing

#### Fidings:
```
quest-protocol\contracts\RabbitHoleReceipt.sol::178 => function royaltyInfo(
quest-protocol\contracts\RabbitHoleTickets.sol::109 => function royaltyInfo(
```

#### Tools used
Manual

### [N05] Expressions for constant values such as a call to keccak256(), should use immutabel rather than constant

#### Findings:
```
quest-protocol\contracts\QuestFactory.sol::17 => bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
```

#### Tools used
Manual

### [N06] Unused import

#### Findings:
```
quest-protocol\contracts\Quest.sol::7 => import {ECDSA} from '@openzeppelin/contracts/utils/cryptography/ECDSA.sol';
```

#### Tools used
Manual

### [N07] Constants should be defined rather than magic numbers

#### Findings:
```
quest-protocol\contracts\Erc20Quest.sol::53 => return (maxTotalRewards() * questFee) / 10_000;
quest-protocol\contracts\Erc20Quest.sol::97 => return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
quest-protocol\contracts\QuestFactory.sol::187 => if (questFee_ > 10_000) revert QuestFeeTooHigh();
quest-protocol\contracts\RabbitHoleReceipt.sol::184 => uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
quest-protocol\contracts\RabbitHoleTickets.sol::113 => uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

#### Tools used
Manual

### [N08] Override function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings

#### Findings:
```
quest-protocol\contracts\RabbitHoleTickets.sol::109 => function royaltyInfo(
```

#### Tools used
Manual