
## 01 Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions

See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

_There are 3 instances of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
File: contracts/QuestFactory.sol

16: contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
File: contracts/RabbitHoleReceipt.sol

15-21: contract RabbitHoleReceipt is
    Initializable,
    ERC721Upgradeable,
    ERC721EnumerableUpgradeable,
    ERC721URIStorageUpgradeable,
    OwnableUpgradeable,
    IERC2981Upgradeable
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
File: contracts/RabbitHoleTickets.sol

11-16: contract RabbitHoleTickets is
    Initializable,
    ERC1155Upgradeable,
    OwnableUpgradeable,
    ERC1155BurnableUpgradeable,
    IERC2981Upgradeable
```

----

## 02 Missing events for functions that change critical parameters 

The afunctions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

Missing events and timelocks do not promote transparency and if such changes immediately affect users’ perception of fairness or trustworthiness, they could exit the protocol causing a reduction in liquidity which could negatively impact protocol TVL and reputation.

### Recommended Mitigation Steps

Add events to all functions that change critical parameters.

_There are 16 instances of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

```
File: contracts/Quest.sol

50: function start() public virtual onlyOwner {
57: function pause() public onlyOwner onlyStarted {
63: function unPause() public onlyOwner onlyStarted {
150: function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
File: contracts/Erc20Quest.sol

81: function withdrawRemainingTokens(address to_) public override onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
File: contracts/QuestFactory.sol

142: function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
159: function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
165: function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner
172: function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
179: function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
186: function setQuestFee(uint256 questFee_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
File: contracts/RabbitHoleReceipt.sol

65: function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
71: function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
77: function setQuestFactory(address questFactory_) public onlyOwner {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
File: contracts/RabbitHoleTickets.sol

54: function setTicketRenderer(address ticketRenderer_) public onlyOwner {
60: function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
```

---------

## 03 Using both named returns and a return statement isn't necessary 

Removing unused named returns variables can reduce gas usage (MSTOREs/MLOADs) and improve code clarity. To save gas and improve code quality: consider using only one of those.

_There are 2 instances of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
File: contracts/RabbitHoleReceipt.sol

178: function royaltyInfo(
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
File: contracts/RabbitHoleTickets.sol

109: function royaltyInfo(
```

------

## 04 Constants should be defined rather than using magic numbers

_There are 5 instances of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
File: contracts/QuestFactory.sol

48: setQuestFee(2_000);
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
File: contracts/Erc20Quest.sol

53: return (maxTotalRewards() * questFee) / 10_000;
97: return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
File: contracts/RabbitHoleReceipt.sol

184: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
File: contracts/RabbitHoleTickets.sol

113: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

----

## 05 Large multiples of ten should use scientific notation

Use (e.g. 1e12) rather than decimal literals (e.g. 1000000000000), for better code readability

_There are 4 instances of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
File: contracts/Erc20Quest.sol

53: return (maxTotalRewards() * questFee) / 10_000;
97: return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
File: contracts/RabbitHoleReceipt.sol

184: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
File: contracts/RabbitHoleTickets.sol

113: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

---------

## 06 Open TODOS

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

_There is 1 instance of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol

```
File: contracts/interfaces/IQuest.sol

4: // TODO clean this whole thing up
```

---------

## 07 Use named imports instead of plain 'import file.sol'

For instance, you use regular imports such as:

[QuestFactory.sol#L4](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L4)

```
import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
```

Instead of this, use named imports such as:

```
import {Initializable.sol} from '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
```

_There are 23 instances of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
File: contracts/QuestFactory.sol

4: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
10: import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol';
11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
File: contracts/RabbitHoleReceipt.sol

4: import '@openzeppelin/contracts-upgradeable/token/ERC721/ERC721Upgradeable.sol';
5: import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721URIStorageUpgradeable.sol';
6: import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721EnumerableUpgradeable.sol';
7: import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
8: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
9: import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';
10: import '@openzeppelin/contracts-upgradeable/utils/CountersUpgradeable.sol';
11: import './ReceiptRenderer.sol';
12: import './interfaces/IQuestFactory.sol';
13: import './interfaces/IQuest.sol';
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
File: contracts/RabbitHoleTickets.sol

4: import '@openzeppelin/contracts-upgradeable/token/ERC1155/ERC1155Upgradeable.sol';
5: import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
6: import '@openzeppelin/contracts-upgradeable/token/ERC1155/extensions/ERC1155BurnableUpgradeable.sol'
7: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
8: import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';
9: import './TicketRenderer.sol';
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol

```
File: contracts/ReceiptRenderer.sol

4: import '@openzeppelin/contracts/utils/Base64.sol';
5: import '@openzeppelin/contracts/utils/Strings.sol';
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol

```
File: contracts/TicketRenderer.sol

4: import '@openzeppelin/contracts/utils/Base64.sol';
5: import '@openzeppelin/contracts/utils/Strings.sol';
```

------

## 08 Include return parameters in natspec comment

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

[https://docs.soliditylang.org/en/v0.8.15/natspec-format.html](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

If Return parameters are declared, you must prefix them with `/// @return`.

_There are 9 instances of this issue_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
File: contracts/QuestFactory.sol

193: function getNumberMinted(string memory questId_) external view returns (uint) {
210: function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
File: contracts/RabbitHoleReceipt.sol

109-112: function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory) {

158-160: function tokenURI(
        uint tokenId_
    ) public view virtual override(ERC721Upgradeable, ERC721URIStorageUpgradeable) returns (string memory) {

178-181: function royaltyInfo(
        uint256 tokenId_,
        uint256 salePrice_
    ) external view override returns (address receiver, uint256 royaltyAmount) {
    
190-192: function supportsInterface(
        bytes4 interfaceId_
    ) public view virtual override(ERC721Upgradeable, ERC721EnumerableUpgradeable, IERC165Upgradeable) returns (bool) {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
File: contracts/RabbitHoleTickets.sol

109-112: function royaltyInfo(
        uint256 tokenId_,
        uint256 salePrice_
    ) external view override returns (address receiver, uint256 royaltyAmount) {

119-121: function supportsInterface(
        bytes4 interfaceId_
    ) public view virtual override(ERC1155Upgradeable, IERC165Upgradeable) returns (bool) {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol

```
File: contracts/ReceiptRenderer.sol

40-47: function generateDataURI(
        uint tokenId_,
        string memory questId_,
        uint totalParticipants_,
        bool claimed_,
        uint rewardAmount_,
        address rewardAddress_
    ) public view virtual returns (bytes memory) {

82: function generateAttribute(string memory key, string memory value) public pure returns (string memory) {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
File: contracts/Erc20Quest.sol

96: function protocolFee() public view returns (uint256) {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

```
File: contracts/Quest.sol

122: function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {
135: function isClaimed(uint256 tokenId_) public view returns (bool) {
140: function getRewardAmount() public view returns (uint256) {
145: function getRewardToken() public view returns (address) {
```

----

## 09 Use a more recent version of solidity

It's a best practice to use the latest compiler version.

The specified minimum compiler version is quite old. Older compilers might be susceptible to some bugs. We recommend changing the solidity version pragma to the latest version to enforce the use of an up to date compiler.

List of known compiler bugs and their severity can be found here: [https://etherscan.io/solcbuginfo](https://etherscan.io/solcbuginfo)

_This issue exists in all the In-scope contracts_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

```
File: contracts/Quest.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
File: contracts/Erc20Quest.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol

```
File: contracts/Erc1155Quest.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
File: contracts/QuestFactory.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
File: contracts/RabbitHoleReceipt.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
File: contracts/RabbitHoleTickets.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol

```
File: contracts/ReceiptRenderer.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol

```
File: contracts/TicketRenderer.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol

```
File: contracts/interfaces/IQuest.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol

```
File: contracts/interfaces/IQuestFactory.sol

2: pragma solidity ^0.8.15;
```

----------

## 10 Non-library or interface files should use fixed compiler versions, not floating ones

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

_This issue exists in all the In-scope contracts_

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol

```
File: contracts/Quest.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol

```
File: contracts/Erc20Quest.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol

```
File: contracts/Erc1155Quest.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol

```
File: contracts/QuestFactory.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol

```
File: contracts/RabbitHoleReceipt.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol

```
File: contracts/RabbitHoleTickets.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol

```
File: contracts/ReceiptRenderer.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol

```
File: contracts/TicketRenderer.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol

```
File: contracts/interfaces/IQuest.sol

2: pragma solidity ^0.8.15;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol

```
File: contracts/interfaces/IQuestFactory.sol

2: pragma solidity ^0.8.15;
```

-----------


