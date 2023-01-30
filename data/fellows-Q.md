# Low Risk Issue List

| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [L-01] | There is a risk that the `questFee` variable is accidentally initialized  or set to 0 and platform loses money| 1 |
| [L-02] |Important parameter changes should emit events| 3 |
| [L-03] | Use two step ownership change| 4 |
| [L-04] | Owner can renounce Ownership| 4 |

Total 4 issues

## [L-01] There is a risk that the `questFee` variable is accidentally initialized  or set to 0 and platform loses money

```
1 result - 1 file

quest-protocol/contracts/QuestFactory.sol:
  200:     function setQuestFee(uint256 questFee_) public onlyOwner {
  201          if (questFee_ > 10_000) revert QuestFeeTooHigh();
```

A user can accidentally set the quest fee to zero on initialization or within the setter. The documentation does not specify whether setting the fee to zero is allowed. Presumably, the protocol would like to enforce the fee, so they make money. 

## [L-02] Important parameter changes should emit events

```
10 results - 3 files

quest-protocol/contracts/QuestFactory.sol:
  169:     function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
  170          claimSignerAddress = claimSignerAddress_;

  174      /// @param protocolFeeRecipient_ The address of the protocol fee recipient
  175:     function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
  176          if (protocolFeeRecipient_ == address(0)) revert AddressZeroNotAllowed();

  183:     function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
  184          rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);

  191:     function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
  192          rewardAllowlist[rewardAddress_] = allowed_;

  200:     function setQuestFee(uint256 questFee_) public onlyOwner {
  201          if (questFee_ > 10_000) revert QuestFeeTooHigh();

quest-protocol/contracts/RabbitHoleReceipt.sol:
   86      /// @param receiptRenderer_ the address of the receipt renderer contract
   87:     function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
   88          ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);

   92      /// @param royaltyRecipient_ the address of the royalty recipient
   93:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
   94          royaltyRecipient = royaltyRecipient_;

   98      /// @param questFactory_ the address of the quest factory contract
   99:     function setQuestFactory(address questFactory_) public onlyOwner {
  100          QuestFactoryContract = IQuestFactory(questFactory_);

quest-protocol/contracts/RabbitHoleTickets.sol:
  56      /// @param ticketRenderer_ the address of the ticket renderer contract
  57:     function setTicketRenderer(address ticketRenderer_) public onlyOwner {
  58          TicketRendererContract = TicketRenderer(ticketRenderer_);

  62      /// @param royaltyRecipient_ the address of the royalty recipient
  63:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
  64          royaltyRecipient = royaltyRecipient_;
```

Events are generally emitted when sensitive information changes in the contract. 

**Recommendation:** Emit events when important variables are changed. 

## L-03 Use two step ownership change

```
13 results - 4 files

quest-protocol/contracts/Quest.sol:
   3  
   4: import {Ownable} from '@openzeppelin/contracts/access/Ownable.sol';
   5  import {IQuest} from './interfaces/IQuest.sol';

  13: contract Quest is Ownable, IQuest {
  14      RabbitHoleReceipt public immutable rabbitHoleReceiptContract;

quest-protocol/contracts/QuestFactory.sol:
  10  import {RabbitHoleReceipt} from './RabbitHoleReceipt.sol';
  11: import {OwnableUpgradeable} from '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
  12  import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol';

  19: contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory { 

  47      ) public initializer {
  48:         __Ownable_init();
  49          __AccessControl_init();

quest-protocol/contracts/RabbitHoleReceipt.sol:
   6  import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721EnumerableUpgradeable.sol';
   7: import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
   8  import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';

  29      ERC721URIStorageUpgradeable,
  30:     OwnableUpgradeable,
  31      IERC2981Upgradeable

  62          __ERC721URIStorage_init();
  63:         __Ownable_init();
  64          royaltyRecipient = royaltyRecipient_;

quest-protocol/contracts/RabbitHoleTickets.sol:
   4  import '@openzeppelin/contracts-upgradeable/token/ERC1155/ERC1155Upgradeable.sol';
   5: import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
   6  import '@openzeppelin/contracts-upgradeable/token/ERC1155/extensions/ERC1155BurnableUpgradeable.sol';

  15      ERC1155Upgradeable,
  16:     OwnableUpgradeable,
  17      ERC1155BurnableUpgradeable,

  40          __ERC1155_init('');
  41:         __Ownable_init();
  42          __ERC1155Burnable_init();
```

Accidentally setting the wrong address will lead to a complete loss of control on the protocol.

**Recommendation:** Implement a two-step process where a new owner is nominated and then must accept ownership. 

## L-04 Owner can renounce Ownership

```
13 results - 4 files

quest-protocol/contracts/Quest.sol:
   3  
   4: import {Ownable} from '@openzeppelin/contracts/access/Ownable.sol';
   5  import {IQuest} from './interfaces/IQuest.sol';

  13: contract Quest is Ownable, IQuest {
  14      RabbitHoleReceipt public immutable rabbitHoleReceiptContract;

quest-protocol/contracts/QuestFactory.sol:
  10  import {RabbitHoleReceipt} from './RabbitHoleReceipt.sol';
  11: import {OwnableUpgradeable} from '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
  12  import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol';

  19: contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory { 

  47      ) public initializer {
  48:         __Ownable_init();
  49          __AccessControl_init();

quest-protocol/contracts/RabbitHoleReceipt.sol:
   6  import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721EnumerableUpgradeable.sol';
   7: import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
   8  import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';

  29      ERC721URIStorageUpgradeable,
  30:     OwnableUpgradeable,
  31      IERC2981Upgradeable

  62          __ERC721URIStorage_init();
  63:         __Ownable_init();
  64          royaltyRecipient = royaltyRecipient_;

quest-protocol/contracts/RabbitHoleTickets.sol:
   4  import '@openzeppelin/contracts-upgradeable/token/ERC1155/ERC1155Upgradeable.sol';
   5: import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
   6  import '@openzeppelin/contracts-upgradeable/token/ERC1155/extensions/ERC1155BurnableUpgradeable.sol';

  15      ERC1155Upgradeable,
  16:     OwnableUpgradeable,
  17      ERC1155BurnableUpgradeable,

  40          __ERC1155_init('');
  41:         __Ownable_init();
  42          __ERC1155Burnable_init();
```

The Ownable contract comes with built in functionality to renounce ownership, making it impossible to administer the contracts. In some cases, this is the desired outcome. However, the decision should be made and clearly communicated. 

**Recommendation:** Either remove the functionality entirely, or clearly document that the protocol intended to renounce ownership at some point in the future. 


# Non-Critical Issue List

| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01]|Mixed-use of uint and uint256 |37|
| [N-02] |Initial value check is missing in Set Functions  |15|
| [N-03] |`Function writing` that does not comply with the `Solidity Style Guide` |All Contracts|
| [N-04] |Missing NatSpec return statement|13|
| [N-05] |Solidity compiler optimizations can be problematic|1|
| [N-06] |Insufficient Code Coverage|2|

Total 6 issues

## [N-01] Mixed-use of uint and uint256

Throughout the project, uint256 and uint is used interchangeably within the same files. It is best practices to choose one and consistently use it throughout the project. Generally, uint256 is preferred and is most prevalently used throughout the project. 

Particularly, a developer can misinterpret the intent in `ReceiptRenderer.sol` and `TicketRenderer.sol` as the protocol implements `using Strings for uint256;` 

```
37 results - 8 files

quest-protocol/contracts/Erc20Quest.sol:
  87  
  88:         uint unclaimedTokens = (receiptRedeemers() - redeemedTokens) * rewardAmountInWeiOrTokenId;
  89          uint256 nonClaimableTokens = IERC20(rewardToken).balanceOf(address(this)) - protocolFee() - unclaimedTokens;

quest-protocol/contracts/Quest.sol:
   73      function _setClaimed(uint256[] memory tokenIds_) private {
   74:         for (uint i = 0; i < tokenIds_.length; i++) {
   75              claimedList[tokenIds_[i]] = true;

  113          uint256 redeemableTokenCount = 0;
  114:         for (uint i = 0; i < tokens.length; i++) {
  115              if (!isClaimed(tokens[i])) {

quest-protocol/contracts/QuestFactory.sol:
  26          address questAddress;
  27:         uint totalParticipants;
  28:         uint numberMinted;
  29      }

  35      mapping(address => bool) public rewardAllowlist;
  36:     uint public questFee;
  37:     uint public questIdCount;
  38  

quest-protocol/contracts/RabbitHoleReceipt.sol:
   41:     mapping(uint => string) public questIdForTokenId;
   42      address public royaltyRecipient;
   43      address public minterAddress;
   44:     uint public royaltyFee;
   45:     mapping(uint => uint) public timestampForTokenId;
   46      ReceiptRenderer public ReceiptRendererContract;

   58          address minterAddress_,
   59:         uint royaltyFee_
   60      ) public initializer {

  123          _tokenIds.increment();
  124:         uint newTokenID = _tokenIds.current();
  125          questIdForTokenId[newTokenID] = questId_;

  136      ) public view returns (uint[] memory) {
  137:         uint msgSenderBalance = balanceOf(claimingAddress_);
  138          uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
  139:         uint foundTokens = 0;
  140  
  141:         for (uint i = 0; i < msgSenderBalance; i++) {
  142:             uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
  143              if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {

  149          uint[] memory filteredTokens = new uint[](foundTokens);
  150:         uint filterTokensIndexTracker = 0;
  151  
  152:         for (uint i = 0; i < msgSenderBalance; i++) {
  153              if (tokenIdsForQuest[i] > 0) {

  182      function tokenURI(
  183:         uint tokenId_
  184      ) public view virtual override(ERC721Upgradeable, ERC721URIStorageUpgradeable) returns (string memory) {

  188          string memory questId = questIdForTokenId[tokenId_];
  189:         (address questAddress, uint totalParticipants, ) = QuestFactoryContract.questInfo(questId);
  190          IQuest questContract = IQuest(questAddress);

  192          bool claimed = questContract.isClaimed(tokenId_);
  193:         uint rewardAmount = questContract.getRewardAmount();
  194          address rewardAddress = questContract.getRewardToken();

quest-protocol/contracts/RabbitHoleTickets.sol:
   25      address public minterAddress;
   26:     uint public royaltyFee;
   27      TicketRenderer public TicketRendererContract;

   37          address minterAddress_,
   38:         uint royaltyFee_
   39      ) public initializer {

  104      /// @dev return the uri, this delegates to the ticket renderer contract
  105:     function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {
  106          return TicketRendererContract.generateTokenURI(tokenId_);

quest-protocol/contracts/ReceiptRenderer.sol:
   26      function generateTokenURI(
   27:         uint tokenId_,
   28          string memory questId_,
   29:         uint totalParticipants_,
   30          bool claimed_,
   31:         uint rewardAmount_,
   32          address rewardAddress_

   45      function generateDataURI(
   46:         uint tokenId_,
   47          string memory questId_,
   48:         uint totalParticipants_,
   49          bool claimed_,
   50:         uint rewardAmount_,
   51          address rewardAddress_

  104      /// @return encoded JSON for an SVG image
  105:     function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {
  106          bytes memory svg = abi.encodePacked(

quest-protocol/contracts/TicketRenderer.sol:
  12  
  13:     Best to change uint to uint256 to more explicitly communicate to devs 
  14      since you are using Strings for uint256

  23      function generateTokenURI(
  24:         uint tokenId_
  25      ) public pure returns (string memory) {

  42      /// @return encoded JSON for an SVG image
  43:     function generateSVG(uint tokenId_) public pure returns (string memory) {
  44          bytes memory svg = abi.encodePacked(

quest-protocol/contracts/test/TestERC20.sol:
   6  contract TestERC20 is ERC20 {
   7:     constructor(string memory name_, string memory symbol_, uint amountToMint) ERC20(name_, symbol_) {
   8          setBalance(msg.sender, amountToMint);

  12      // this mints/burns the amount depending on the current balance
  13:     function setBalance(address to, uint amount) public {
  14:         uint old = balanceOf(to);
  15          if (old < amount) {

```

## [N-02] Initial value check is missing in Set Functions

Check whether new value is equal to old value in order to prevent unnecessary storage writes. 

```
15 results - 4 files

quest-protocol/contracts/QuestFactory.sol:
  168:     function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
  169          claimSignerAddress = claimSignerAddress_;

  173      /// @param protocolFeeRecipient_ The address of the protocol fee recipient
  174:     function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
  175          if (protocolFeeRecipient_ == address(0)) revert AddressZeroNotAllowed();

  182:     function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
  183          rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);

  190:     function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
  191          rewardAllowlist[rewardAddress_] = allowed_;

  199:     function setQuestFee(uint256 questFee_) public onlyOwner {
  200          if (questFee_ > 10_000) revert QuestFeeTooHigh();

quest-protocol/contracts/RabbitHoleReceipt.sol:
   86      /// @param receiptRenderer_ the address of the receipt renderer contract
   87:     function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
   88          ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);

   92      /// @param royaltyRecipient_ the address of the royalty recipient
   93:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
   94          royaltyRecipient = royaltyRecipient_;

   98      /// @param questFactory_ the address of the quest factory contract
   99:     function setQuestFactory(address questFactory_) public onlyOwner {
  100          QuestFactoryContract = IQuestFactory(questFactory_);

  104      /// @param minterAddress_ the address of the minter
  105:     function setMinterAddress(address minterAddress_) public onlyOwner {
  106          minterAddress = minterAddress_;

  113:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
  114          royaltyFee = royaltyFee_;

quest-protocol/contracts/RabbitHoleTickets.sol:
  56      /// @param ticketRenderer_ the address of the ticket renderer contract
  57:     function setTicketRenderer(address ticketRenderer_) public onlyOwner {
  58          TicketRendererContract = TicketRenderer(ticketRenderer_);

  62      /// @param royaltyRecipient_ the address of the royalty recipient
  63:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
  64          royaltyRecipient = royaltyRecipient_;

  68      /// @param royaltyFee_ the royalty fee
  69:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
  70          royaltyFee = royaltyFee_;

  75      /// @param minterAddress_ the address of the minter
  76:     function setMinterAddress(address minterAddress_) public onlyOwner {
  77          minterAddress = minterAddress_;

quest-protocol/contracts/test/TestERC20.sol:
  12      // this mints/burns the amount depending on the current balance
  13:     function setBalance(address to, uint amount) public {
  14          uint old = balanceOf(to);
```

## [N-03] Function writing` that does not comply with the `Solidity Style Guide

The order of functions helps developers understand the codebase easier. We recommend adhering to the Solidity Style Guide for function ordering. https://docs.soliditylang.org/en/v0.8.17/style-guide.html

This order is recommended: 
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last

Also, it is best practice to begin an internal function with an underscore. See `QuestFactory.sol::grantDefaultAdminAndCreateQuestRole`. 

## [N-04] Missing NatSpec return statement

Providing clear NatSpec will make it easier for developers and users interacting with the protocol to more clearly understand the intent of functions. 

The project is missing NatSpec for return for the following functions: 

- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L95
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L134
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L139
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L144
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L193
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L199
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L209
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L108
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L157
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L177
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L101
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L108
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L118

## [N-05] Solidity compiler optimizations can be problematic

```
const config: HardhatUserConfig = {
  gasReporter: {
    gasPrice: 100,
  },
  defender: {
    apiKey: process.env.DEFENDER_TEAM_API_KEY,
    apiSecret: process.env.DEFENDER_TEAM_API_SECRET_KEY,
  },
  solidity: {
    compilers: [
      {
        version: '0.8.15',
        settings: {
          optimizer: {
            enabled: true,
            runs: 5000,
          },
        },
      },
    ],
  },
```

There have been multiple exploits due to compiler optimizations and it is unclear what issues may occur in the future. 

It is recommended to weigh the pros and cons of gas savings using optimizations against potential exploits. 

## [N-06] Insufficient code coverage

[Code coverage screenshot](https://share.getcloudapp.com/6quNANYW)

It is recommended to strive for 100% test coverage. Particularly, adding more test coverage in `RabitHoleReceipt.sol` and `RabitHoleTickets.sol` as coverage is <65% in those files. 
