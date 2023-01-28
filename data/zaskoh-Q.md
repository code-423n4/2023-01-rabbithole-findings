## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| L-01 | Follow the suggested Proxy pattern for TransparentUpgradeableProxy in QuestFactory | 1 |
| L-02 | Emit event after updating a important state variable by the admin | 10 |
| L-03 | Check fee adjustments before updating state | 2 |
| L-04 | Check if token exists before calculating the royalty | 1 |
| L-05 | Rename missleading modifier name | 1 |
| L-06 | Move questFactoryContract from Erc20Quest to base Quest | 1 |
| L-07 | onlyMinter in RabbitHoleReceipt and RabbitHoleTickets should check for factory | 2 |
| L-08 | Creation of Erc1155Quest's is missing a check and needs additional privileged access | 1 |

**Total: 19 instances over 8 issues**


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| NC-01 | QuestFactory.changeCreateQuestRole is front-runnable | 1 |
| NC-02 | Reward-tokens with fees on transfer will not work | 1 |
| NC-03 | Error message is missleading | 2 |
| NC-04 | Add missing checks in Quest constructor | 3 |
| NC-05 | Quests can be started before startTime | 1 |
| NC-06 | Remove unneded getter functions for public state variables | 2 |
| NC-07 | State variable can't be checked off-chain | 1 |
| NC-08 | Consider making Quest abstract | 1 |
| NC-09 | Complexity can be removed through removing duplicate code | 1 |
| NC-10 | Resolve unused function parameter compiler warnings | 3 |
| NC-11 | Defined return value not used | 2 |
| NC-12 | Unnecessarily imports | 1 |
| NC-13 | Use specific imports instead of just a global import | 23 |
| NC-14 | Solidity pragma versioning | 8 |

**Total: 50 instances over 14 issues**


### Informational
| |Issue|Instances|
|-|:-|:-:|
| I-01 | Spelling of variable names | 4 |
| I-02 | Constants should be defined rather than using magic numbers | 5 |
| I-03 | Don't mix uint and uint256 | 33 |
| I-04 | Long lines | 2 |
| I-05 | Typos | 1 |
| I-06 | Missing documentation | 21 |
| I-07 | Resolve todo in IQuest.sol | 1 |

**Total: 67 instances over 7 issues**

---

## Low Risk Issues

### L-01 Follow the suggested Proxy pattern for TransparentUpgradeableProxy in QuestFactory
[Openzepplin](https://docs.openzeppelin.com/contracts/4.x/api/proxy#Initializable-_disableInitializers-- ) recommends using `_disableInitializers();` since 4.6.0 instead of the modifier `initilizer`. 

Currently you have a constructor with the `initilizer` modifier in QuestFactory. You should change it to use `_disableInitializers();` inside the constructor to follow the suggested implementation from Openzepplin.

```solidity
File: contracts/QuestFactory.sol
35:     constructor() initializer {}
```

### L-02 Emit event after updating a important state variable by the admin
After updating an important state variable from the admin you should emit an event with the update, so it's easier to get notified off-chain if for example an important destination contract, the fees are changed after an admin update.

```solidity
File: contracts/QuestFactory.sol
159:     function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
160:         claimSignerAddress = claimSignerAddress_;
161:     }

File: contracts/QuestFactory.sol
165:     function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
166:         if (protocolFeeRecipient_ == address(0)) revert AddressZeroNotAllowed();
167:         protocolFeeRecipient = protocolFeeRecipient_;
168:     }

File: contracts/QuestFactory.sol
172:     function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
173:         rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
174:     }

File: contracts/QuestFactory.sol
179:     function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
180:         rewardAllowlist[rewardAddress_] = allowed_;
181:     }

File: contracts/QuestFactory.sol
186:     function setQuestFee(uint256 questFee_) public onlyOwner {
187:         if (questFee_ > 10_000) revert QuestFeeTooHigh();
188:         questFee = questFee_;
189:     }

File: contracts/RabbitHoleReceipt.sol
65:     function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
66:         ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);
67:     }

File: contracts/RabbitHoleReceipt.sol
71:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
72:         royaltyRecipient = royaltyRecipient_; 
73:     }

File: contracts/RabbitHoleReceipt.sol
77:     function setQuestFactory(address questFactory_) public onlyOwner {
78:         QuestFactoryContract = IQuestFactory(questFactory_);
79:     }

File: contracts/RabbitHoleTickets.sol
54:     function setTicketRenderer(address ticketRenderer_) public onlyOwner {
55:         TicketRendererContract = TicketRenderer(ticketRenderer_);
56:     }

File: contracts/RabbitHoleTickets.sol
60:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
61:         royaltyRecipient = royaltyRecipient_;
62:     }

```

### L-03 Check fee adjustments before updating state
Before updating fees you should check at least that the new value is not greater than X. Contract can get in a bad state if the fee is too high.

```solidity
File: contracts/RabbitHoleTickets.sol
66:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
67:         royaltyFee = royaltyFee_;
68:         emit RoyaltyFeeSet(royaltyFee_);
69:     }

File: contracts/RabbitHoleReceipt.sol
90:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
91:         royaltyFee = royaltyFee_;
92:         emit RoyaltyFeeSet(royaltyFee_);
93:     }

```

### L-04 Check if token exists before calculating the royalty
At the moment the `RabbitHoleTickets.royaltyInfo` function is not checking if the token exists. You should consider adding a check if the token exists before you calculate the amount, or it can lead to unexpected behaviour for calling contracts.

```solidity
File: contracts/RabbitHoleTickets.sol
109:     function royaltyInfo(
110:         uint256 tokenId_,
111:         uint256 salePrice_
112:     ) external view override returns (address receiver, uint256 royaltyAmount) {
113:         uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
114:         return (royaltyRecipient, royaltyPayment);
115:     }

```

### L-05 Rename missleading modifier name
The modifier `onlyAdminWithdrawAfterEnd` in Quest is missleading. It suggests that there is a Admin check, but it only checks for the endTime.  
Rename the modifier to `onlyWithdrawAfterEnd` or add the admin check.

```solidity
File: contracts/Quest.sol
76:     modifier onlyAdminWithdrawAfterEnd() {
77:         if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
78:         _;
79:     }

```

### L-06 Move questFactoryContract from Erc20Quest to base Quest
At the moment the factory address is only stored in an Erc20Quest but it should be also accessible in the Erc1155Quest as a genral speaking, a Quest is most likely always connected to the deploying factory contract. It's very likely that the factory will also be needed in the Erc1155Quest later on, so it should be moved in the base contract Quest.

```solidity
File: contracts/Erc20Quest.sol
40:         questFactoryContract = QuestFactory(msg.sender);
```

### L-07 onlyMinter in RabbitHoleReceipt and RabbitHoleTickets should check for factory
Currently RabbitHoleReceipt and RabbitHoleTickets check in the modifier that the caller is the minterAddress. This means, that the minter can be different to the QuestFactory and lead to unexpected behaviour.

You should consider checking for `msg.sender == address(QuestFactoryContract)` as it's always the factory that should be allowed to mint. With this you take out complexity and remove a potential breach or miss-configuration if address(QuestFactoryContract) != minterAddress.

```solidity
File: contracts/RabbitHoleReceipt.sol
58:     modifier onlyMinter() {
59:         msg.sender == minterAddress;
60:         _;
61:     }

File: contracts/RabbitHoleTickets.sol
47:     modifier onlyMinter() {
48:         msg.sender == minterAddress;
49:         _;
50:     }

```

### L-08 Creation of Erc1155Quest's is missing a check and needs additional privileged access
Currently in a creation for a new Erc1155Quest it's not checked if the reward-token is approved and also it needs the caller to be the owner.

As it requires the caller to be also the owner, it assume that the owner will always have the CREATE_QUEST_ROLE. You should consider if it's not better to seperate the owner of the contract from the priviliged user that can create a Erc1155Quest. You could split up the create quest role in two roles, one for ERC20 and one for ERC1155.

```solidity
File: contracts/QuestFactory.sol
// restricts for only CREATE_QUEST_ROLE
69:     ) public onlyRole(CREATE_QUEST_ROLE) returns (address) { 
105:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
// missing the check if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
// requires owner to have CREATE_QUEST_ROLE
106:             if (msg.sender != owner()) revert OnlyOwnerCanCreate1155Quest(); 
```

---

## Non-critical Issues

### NC-01 QuestFactory.changeCreateQuestRole is front-runnable
The current implementation of `QuestFactory.changeCreateQuestRole` is front-runnable by a malicious actor with the CREATE_QUEST_ROLE role. He could spam the system with lots of new "official" creations of quests before the role will be revoked.

```solidity
File: contracts/QuestFactory.sol
142:     function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
143:         if (canCreateQuest_) {
144:             _grantRole(CREATE_QUEST_ROLE, account_);
145:         } else {
146:             _revokeRole(CREATE_QUEST_ROLE, account_);
147:         }
148:     }
```

### NC-02 Reward-tokens with fees on transfer will not work
The current implementation and calculations don't allow tokens with fees on transfer. You should be careful by adding such a token to the allow-list.  
Double-check before adding a new token to the allowList, so it's not breaking the expected behaviour.

```solidity
File: contracts/QuestFactory.sol
179:     function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
180:         rewardAllowlist[rewardAddress_] = allowed_; 
181:     }
```

### NC-03 Error message is missleading
Currently you use the error `revert TotalAmountExceedsBalance();` in Erc20Quest and Erc1155Quest in the start function to inform that it can't be started as the needed minimum amount is not in the contract.

Consider renaming the error to something like `NotEnoughBalanceToStart()`.

```solidity
File: contracts/Erc20Quest.sol
58:     function start() public override {
59:         if (IERC20(rewardToken).balanceOf(address(this)) < maxTotalRewards() + maxProtocolReward())
60:             revert TotalAmountExceedsBalance();
61:         super.start();
62:     }

File: contracts/Erc1155Quest.sol
33:     function start() public override {
34:         if (IERC1155(rewardToken).balanceOf(address(this), rewardAmountInWeiOrTokenId) < totalParticipants)
35:             revert TotalAmountExceedsBalance();
36:         super.start();
37:     }
```

### NC-04 Add missing checks in Quest constructor
The current constructor in Quest.sol is missing some important checks that can lead to a bad state in the deployed contract.

1. There is no check for a minimum run-time. Currently it's possible that a quest will only run 1 second.
2. totalParticipants can be 0 and will break calculations. It makes no sense to have 0 participants, check for > 0.
3. rewardAmountInWeiOrTokenId can be 0. Check for > 0.

### NC-05 Quests can be started before startTime
At the moment it's possible for the admin to start the quest before the startTime is reached. This is fine as the claim() function has also the onlyQuestActive modifier that checks that the startTime is reached. Still this could lead to unexpected behaviour in the future if it will be forgotten that the contract can get in a state where the quest is started but the startTime is not reached yet.

```solidity
File: contracts/Quest.sol
50:     function start() public virtual onlyOwner {
51:         isPaused = false;
52:         hasStarted = true;
53:     }
```

### NC-06 Remove unneded getter functions for public state variables
If a state variable is public, you don't need to define a getter function for that variable in the implementation.

```solidity
File: contracts/Quest.sol
140:     function getRewardAmount() public view returns (uint256) {
141:         return rewardAmountInWeiOrTokenId;rewardAmountInWeiOrTokenId()
142:     }

File: contracts/Quest.sol
145:     function getRewardToken() public view returns (address) {
146:         return rewardToken;
147:     }

```

### NC-07 State variable can't be checked off-chain
With the current implementation it's not possible to query off-chain if a address already minted for the quest (mapping(address => bool) addressMinted;). Consider adding a function to access this information.

```solidity
File: contracts/QuestFactory.sol
19:     struct Quest {
20:         mapping(address => bool) addressMinted;
21:         address questAddress;
22:         uint totalParticipants;
23:         uint numberMinted;
24:     }
28:     mapping(string => Quest) public quests;

```

### NC-08 Consider making Quest abstract
Currently the Quest contract is not abstract and could be deployed directly, as it is only a base contract and should always be implemented in a final contract you should consider making it abstract. This will also make it more clear that Quest is just the base and needs an implementation.

```solidity
File: contracts/Quest.sol
12: contract Quest is Ownable, IQuest {
```

### NC-09 Complexity can be removed through removing duplicate code
Currently the `QuestFactory.createQuest` function has duplicate code that make it a bit complex and error prone if something needs to be changed.

You should only deploy the new Quest depeding of the type in the condition and move the event emit and setting the variables at the end of the function.

### NC-10 Resolve unused function parameter compiler warnings
The compiler warnings regarding `Warning: Unused function parameter. Remove or comment out the variable name to silence this warning.` should be resolved.

```solidity
Warning: Unused function parameter. Remove or comment out the variable name to silence this warning.
   --> contracts/Quest.sol:122:32:
    |
122 |     function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {
    |                                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Warning: Unused function parameter. Remove or comment out the variable name to silence this warning.
   --> contracts/Quest.sol:129:31:
    |
129 |     function _transferRewards(uint256 amount_) internal virtual {
    |                               ^^^^^^^^^^^^^^^

Warning: Unused function parameter. Remove or comment out the variable name to silence this warning.
   --> contracts/RabbitHoleTickets.sol:110:9:
    |
110 |         uint256 tokenId_,
    |         ^^^^^^^^^^^^^^^^
```

### NC-11 Defined return value not used
If you're not using the variable name in the function declaration you should remove it.

```solidity
File: contracts/RabbitHoleTickets.sol
112:     ) external view override returns (address receiver, uint256 royaltyAmount) {
113:         uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
114:         return (royaltyRecipient, royaltyPayment);
115:     }
116: 

File: contracts/RabbitHoleReceipt.sol
181:     ) external view override returns (address receiver, uint256 royaltyAmount) {
182:         require(_exists(tokenId_), 'Nonexistent token');
183: 
184:         uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
185:         return (royaltyRecipient, royaltyPayment);
186:     }
```

### NC-12 Unnecessarily imports
Quest.sol has a unnecessary import that is never used and can be removed.

```solidity
File: contracts/Quest.sol
7: import {ECDSA} from '@openzeppelin/contracts/utils/cryptography/ECDSA.sol';
```

### NC-13 Use specific imports instead of just a global import
For a better better developer experience it's better to use specific imports instead of just a global import. This helps to have a better overview what is realy needed and helps to have a clearer view of the contract.

```solidity
File: contracts/QuestFactory.sol
4: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
10: import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol';
11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

File: contracts/RabbitHoleReceipt.sol
04: import '@openzeppelin/contracts-upgradeable/token/ERC721/ERC721Upgradeable.sol';
05: import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721URIStorageUpgradeable.sol';
06: import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721EnumerableUpgradeable.sol';
07: import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
08: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
09: import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';
10: import '@openzeppelin/contracts-upgradeable/utils/CountersUpgradeable.sol';
11: import './ReceiptRenderer.sol';
12: import './interfaces/IQuestFactory.sol';
13: import './interfaces/IQuest.sol';

File: contracts/RabbitHoleTickets.sol
4: import '@openzeppelin/contracts-upgradeable/token/ERC1155/ERC1155Upgradeable.sol';
5: import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
6: import '@openzeppelin/contracts-upgradeable/token/ERC1155/extensions/ERC1155BurnableUpgradeable.sol';
7: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
8: import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';
9: import './TicketRenderer.sol';

File: contracts/ReceiptRenderer.sol
4: import '@openzeppelin/contracts/utils/Base64.sol';
5: import '@openzeppelin/contracts/utils/Strings.sol';

File: contracts/TicketRenderer.sol
4: import '@openzeppelin/contracts/utils/Base64.sol';
5: import '@openzeppelin/contracts/utils/Strings.sol';

```

### NC-14 Solidity pragma versioning
It's better to have a fixed version for your implementation contracts. Currently you use ^0.8.15 for your contracts but you can (and should) use the fixed Version 0.8.15 / 17 for them.

For your interfaces you could consider a ^0.8.0 version.

```solidity
File: contracts/Erc20Quest.sol
File: contracts/Erc1155Quest.sol
File: contracts/Quest.sol
File: contracts/QuestFactory.sol
File: contracts/RabbitHoleReceipt.sol
File: contracts/RabbitHoleTickets.sol
File: contracts/ReceiptRenderer.sol
File: contracts/TicketRenderer.sol
```

---

## Informational

### I-01 Spelling of variable names
You should not use an uppercase letter to begin with a variable name, especially if it is for an contract instance that is to similar. Also you should not start with _ for your state variables.

```solidity
File: contracts/RabbitHoleTickets.sol
25:     TicketRenderer public TicketRendererContract;

File: contracts/RabbitHoleReceipt.sol
35:     ReceiptRenderer public ReceiptRendererContract; 

File: contracts/RabbitHoleReceipt.sol
36:     IQuestFactory public QuestFactoryContract;

File: contracts/RabbitHoleReceipt.sol
27:     CountersUpgradeable.Counter private _tokenIds;

```

### I-02 Constants should be defined rather than using magic numbers
It is better to define constants instead of using magic numbers in calculations.

```solidity
File: contracts/Erc20Quest.sol
53:         return (maxTotalRewards() * questFee) / 10_000;

File: contracts/Erc20Quest.sol
97:         return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;

File: contracts/QuestFactory.sol
187:         if (questFee_ > 10_000) revert QuestFeeTooHigh();

File: contracts/RabbitHoleReceipt.sol
184:         uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;

File: contracts/RabbitHoleTickets.sol
113:         uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

### I-03 Don't mix uint and uint256
For better readbility and easier identification it's best to use uintX for specific uints. Instead of uint you should stick to uint256 or at least try to stay with one variant per file.

```solidity
File: contracts/Erc20Quest.sol
84:         uint unclaimedTokens = (receiptRedeemers() - redeemedTokens) * rewardAmountInWeiOrTokenId;

File: contracts/Quest.sol
70:         for (uint i = 0; i < tokenIds_.length; i++) {
104:         for (uint i = 0; i < tokens.length; i++) {

File: contracts/QuestFactory.sol
19:     struct Quest {
20:         mapping(address => bool) addressMinted;
21:         address questAddress;
22:         uint totalParticipants;
23:         uint numberMinted;
24:     }

File: contracts/QuestFactory.sol
31:     uint public questFee;
32:     uint public questIdCount;

File: contracts/RabbitHoleReceipt.sol
30:     mapping(uint => string) public questIdForTokenId;
33:     uint public royaltyFee;
34:     mapping(uint => uint) public timestampForTokenId;
47:         uint royaltyFee_
100:         uint newTokenID = _tokenIds.current();
113:         uint msgSenderBalance = balanceOf(claimingAddress_);
115:         uint foundTokens = 0;
117:         for (uint i = 0; i < msgSenderBalance; i++) {
118:             uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
126:         uint filterTokensIndexTracker = 0;
128:         for (uint i = 0; i < msgSenderBalance; i++) {
159:         uint tokenId_
165:         (address questAddress, uint totalParticipants, ) = QuestFactoryContract.questInfo(questId);
169:         uint rewardAmount = questContract.getRewardAmount();

File: contracts/RabbitHoleTickets.sol
24:     uint public royaltyFee;
36:         uint royaltyFee_
102:     function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {

File: contracts/ReceiptRenderer.sol
22:         uint tokenId_,
24:         uint totalParticipants_,
26:         uint rewardAmount_,
41:         uint tokenId_,
43:         uint totalParticipants_,
45:         uint rewardAmount_,
100:     function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {

File: contracts/TicketRenderer.sol
17:         uint tokenId_
36:     function generateSVG(uint tokenId_) public pure returns (string memory) {

```

### I-04 Long lines
Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over [164](https://github.com/aizatto/character-length) characters, the lines below should be split when they reach that length

```solidity
File: contracts/interfaces/IQuestFactory.sol
16:     event QuestCreated(address indexed creator, address indexed contractAddress, string indexed questId, string contractType, address rewardTokenAddress, uint256 endTime, uint256 startTime, uint256 totalParticipants, uint256 rewardAmountOrTokenId);

File: contracts/Erc20Quest.sol
56:     /// @notice Starts the quest by marking it ready to start at the contract level. Marking a quest ready to start does not mean that it is live. It also requires that the start time has passed
57:     /// @dev Requires that the balance of the rewards in the contract is greater than or equal to the maximum amount of rewards that can be claimed by all users and the protocol
58:     function start() public override {
```

### I-05 Typos
There is a typo in a comment.

```solidity
File: contracts/QuestFactory.sol
176:     /// @dev set or remave a contract address to be used as a reward
=> remave != remove
```

### I-06 Missing documentation
The contracts are well documented and clear to read. Still it misses some documentation that could be added for a even better developer experience. Functions missing @param (or wrong param mentioned) / @return / @notice or is missing complete NatSpec.

```solidity
File: contracts/interfaces/IQuest.sol
23:     function isClaimed(uint256 tokenId_) external view returns (bool);
24:     function getRewardAmount() external view returns (uint256);
25:     function getRewardToken() external view returns (address);

File: contracts/interfaces/IQuestFactory.sol
19:     function questInfo(string memory questId_) external view returns (address, uint, uint);

File: contracts/Erc20Quest.sol
96:     function protocolFee() public view returns (uint256) {

File: contracts/Quest.sol
135:     function isClaimed(uint256 tokenId_) public view returns (bool) {

File: contracts/Quest.sol
140:     function getRewardAmount() public view returns (uint256) {

File: contracts/Quest.sol
145:     function getRewardToken() public view returns (address) {

File: contracts/Quest.sol
150:     function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}

File: contracts/QuestFactory.sol
193:     function getNumberMinted(string memory questId_) external view returns (uint) {

File: contracts/QuestFactory.sol
199:     function questInfo(string memory questId_) external view returns (address, uint, uint) {

File: contracts/QuestFactory.sol
210:     function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) {

File: contracts/RabbitHoleReceipt.sol
109:     function getOwnedTokenIdsOfQuest(
110:         string memory questId_,
111:         address claimingAddress_
112:     ) public view returns (uint[] memory) {

File: contracts/RabbitHoleReceipt.sol
158:     function tokenURI(
159:         uint tokenId_
160:     ) public view virtual override(ERC721Upgradeable, ERC721URIStorageUpgradeable) returns (string memory) {

File: contracts/RabbitHoleReceipt.sol
178:     function royaltyInfo(
179:         uint256 tokenId_,
180:         uint256 salePrice_
181:     ) external view override returns (address receiver, uint256 royaltyAmount) {

File: contracts/RabbitHoleReceipt.sol
190:     function supportsInterface(
191:         bytes4 interfaceId_
192:     ) public view virtual override(ERC721Upgradeable, ERC721EnumerableUpgradeable, IERC165Upgradeable) returns (bool) {

File: contracts/RabbitHoleTickets.sol
102:     function uri(uint tokenId_) public view virtual override(ERC1155Upgradeable) returns (string memory) {

File: contracts/RabbitHoleTickets.sol
109:     function royaltyInfo(
110:         uint256 tokenId_,
111:         uint256 salePrice_
112:     ) external view override returns (address receiver, uint256 royaltyAmount) {

File: contracts/RabbitHoleTickets.sol
119:     function supportsInterface(
120:         bytes4 interfaceId_
121:     ) public view virtual override(ERC1155Upgradeable, IERC165Upgradeable) returns (bool) {

File: contracts/ReceiptRenderer.sol
40:     function generateDataURI(
41:         uint tokenId_,
42:         string memory questId_,
43:         uint totalParticipants_,
44:         bool claimed_,
45:         uint rewardAmount_,
46:         address rewardAddress_
47:     ) public view virtual returns (bytes memory) {

File: contracts/ReceiptRenderer.sol
82:     function generateAttribute(string memory key, string memory value) public pure returns (string memory) {
```

### I-07 Resolve todo in IQuest.sol
Before going live you should not have any open todo's in your code. Currently IQuest is commented with a todo. After your have added the missing NatSpec in the file, you should remove the comment.

```solidity
File: contracts/interfaces/IQuest.sol
4: // TODO clean this whole thing up
```