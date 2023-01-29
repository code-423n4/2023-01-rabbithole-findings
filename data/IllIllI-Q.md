
## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | `image_data` should be used for raw svg | 2 | 
| [L&#x2011;02] | Don't use strings for numbers in JSON | 1 | 
| [L&#x2011;03] | Users may self-DOS themselves | 1 | 
| [L&#x2011;04] | `startTime_` should be allowed to be the current block | 1 | 
| [L&#x2011;05] | Upgradeable contract not initialized | 1 | 
| [L&#x2011;06] | Owner can renounce while system is paused | 1 | 
| [L&#x2011;07] | Use `Ownable2Step` rather than `Ownable` | 3 | 
| [L&#x2011;08] | Common code should be a mixin | 1 | 

Total: 11 instances over 8 issues


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | Use `abstract` contracts with unimplemented `virtual` functions | 2 | 
| [N&#x2011;02] | Non-exploitable re-entrancies | 2 | 
| [N&#x2011;03] | `Erc1155Quest` doesn't have an option for a protocol fee | 1 | 
| [N&#x2011;04] | `_disableInitializers()` is preferred for constructors | 1 | 
| [N&#x2011;05] | Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions | 3 | 
| [N&#x2011;06] | Import declarations should import specific identifiers, rather than the whole file | 23 | 
| [N&#x2011;07] | `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings | 1 | 
| [N&#x2011;08] | `constant`s should be defined rather than using magic numbers | 7 | 
| [N&#x2011;09] | Missing event and or timelock for critical parameter change | 5 | 
| [N&#x2011;10] | Events that mark critical parameter changes should contain both the old and the new value | 4 | 
| [N&#x2011;11] | Expressions for constant values such as a call to `keccak256()`, should use `immutable` rather than `constant` | 1 | 
| [N&#x2011;12] | Lines are too long | 3 | 
| [N&#x2011;13] | Non-library/interface files should use fixed compiler versions, not floating ones | 8 | 
| [N&#x2011;14] | Typos | 1 | 
| [N&#x2011;15] | File is missing NatSpec | 2 | 
| [N&#x2011;16] | NatSpec is incomplete | 10 | 
| [N&#x2011;17] | Not using the named return variables anywhere in the function is confusing | 4 | 
| [N&#x2011;18] | Contracts should have full test coverage | 1 | 
| [N&#x2011;19] | Large or complicated code bases should implement fuzzing tests | 1 | 
| [N&#x2011;20] | Function ordering does not follow the Solidity style guide | 9 | 
| [N&#x2011;21] | Contract does not follow the Solidity style guide's suggested layout ordering | 5 | 

Total: 94 instances over 21 issues





## Low Risk Issues

### [L&#x2011;01]  `image_data` should be used for raw svg
`image` is for [urls](https://docs.opensea.io/docs/metadata-standards#metadata-structure). See [this](https://opensea.io/collection/neotokyo-outer-citizens) NFT project which properly uses the `image_data` key, and renders nicely on OpenSea

*There are 2 instances of this issue:*

```solidity
File: /contracts/TicketRenderer.sol

25               '"image": "',
26:              generateSVG(tokenId_),

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L25-L26

```solidity
File: /contracts/ReceiptRenderer.sol

69               '"image": "',
70:              generateSVG(tokenId_, questId_),

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L69-L70

### [L&#x2011;02]  Don't use strings for numbers in JSON
For numbers, OpenSea has [extra](https://docs.opensea.io/docs/metadata-standards#numeric-traits) display capabilities, and different behavior for different `display_type`s. When the field is a number, an number rather than a string should be provided

*There is 1 instance of this issue:*

```solidity
File: /contracts/ReceiptRenderer.sol

58:              generateAttribute('Reward Amount', rewardAmount_.toString()),

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L58

### [L&#x2011;03]  Users may self-DOS themselves
If a user holds too many tokens for a quest, this function may run out of gas, especially since it makes quite a few storage lookups and does external calls in a loop

*There is 1 instance of this issue:*

```solidity
File: /contracts/RabbitHoleReceipt.sol

109      function getOwnedTokenIdsOfQuest(
110          string memory questId_,
111          address claimingAddress_
112:     ) public view returns (uint[] memory) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L109-L112

### [L&#x2011;04]  `startTime_` should be allowed to be the current block
`onlyQuestActive()` allows the timestamp boundary to be on the current block, so it's inconsistent for it to be excluded here

*There is 1 instance of this issue:*

```solidity
File: /contracts/Quest.sol

36:          if (startTime_ <= block.timestamp) revert StartTimeInPast();

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L36

### [L&#x2011;05]  Upgradeable contract not initialized
Upgradeable contracts are initialized via an initializer function rather than by a constructor. Leaving such a contract uninitialized may lead to it being taken over by a malicious user

*There is 1 instance of this issue:*

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit missing __IERC2981_init()
15:   contract RabbitHoleReceipt is

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L15

### [L&#x2011;06]  Owner can renounce while system is paused
The contract owner or single user with a role is not prevented from renouncing the role/ownership while the contract is paused, which would cause any user assets stored in the protocol, to be locked indefinitely

*There is 1 instance of this issue:*

```solidity
File: contracts/Quest.sol

57        function pause() public onlyOwner onlyStarted {
58            isPaused = true;
59:       }

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57-L59

### [L&#x2011;07]  Use `Ownable2Step` rather than `Ownable`
[`Ownable2Step`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/3d7a93876a2e5e1d7fe29b5a0e96e222afdc4cfa/contracts/access/Ownable2Step.sol#L31-L56) and [`Ownable2StepUpgradeable`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/25aabd286e002a1526c345c8db259d57bdf0ad28/contracts/access/Ownable2StepUpgradeable.sol#L47-L63) prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.

*There are 3 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

16:   contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L16

```solidity
File: contracts/RabbitHoleReceipt.sol

15    contract RabbitHoleReceipt is
16        Initializable,
17        ERC721Upgradeable,
18        ERC721EnumerableUpgradeable,
19        ERC721URIStorageUpgradeable,
20        OwnableUpgradeable,
21:       IERC2981Upgradeable

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L15-L21

```solidity
File: contracts/RabbitHoleTickets.sol

11    contract RabbitHoleTickets is
12        Initializable,
13        ERC1155Upgradeable,
14        OwnableUpgradeable,
15        ERC1155BurnableUpgradeable,
16:       IERC2981Upgradeable

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L11-L16

### [L&#x2011;08]  Common code should be a mixin
The code below is repeated for both `RabbitHoleReceipt` and `RabbitHoleTickets`, and instead should be refactored to a common mixin

*There is 1 instance of this issue:*

```solidity
File: /contracts/RabbitHoleReceipt.sol

58       modifier onlyMinter() {
59           msg.sender == minterAddress;
60           _;
61       }
62   
63       /// @dev set the receipt renderer contract
64       /// @param receiptRenderer_ the address of the receipt renderer contract
65       function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
66           ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);
67       }
68   
69       /// @dev set the royalty recipient
70       /// @param royaltyRecipient_ the address of the royalty recipient
71       function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
72           royaltyRecipient = royaltyRecipient_;
73       }
74   
75       /// @dev set the quest factory contract
76       /// @param questFactory_ the address of the quest factory contract
77       function setQuestFactory(address questFactory_) public onlyOwner {
78           QuestFactoryContract = IQuestFactory(questFactory_);
79       }
80   
81       /// @dev set the minter address
82       /// @param minterAddress_ the address of the minter
83       function setMinterAddress(address minterAddress_) public onlyOwner {
84           minterAddress = minterAddress_;
85           emit MinterAddressSet(minterAddress_);
86       }
87   
88       /// @dev set the royalty fee
89       /// @param royaltyFee_ the royalty fee
90       function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
91           royaltyFee = royaltyFee_;
92           emit RoyaltyFeeSet(royaltyFee_);
93:      }

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L58-L93

## Non-critical Issues

### [N&#x2011;01]  Use `abstract` contracts with unimplemented `virtual` functions
The compiler currently warns about the examples below being unreachable code. If you instead change the contracts to be `abstract`, you can leave the functions unimplemented, and force the extending contract to have an implementation

*There are 2 instances of this issue:*

```solidity
File: /contracts/Quest.sol

123:         revert MustImplementInChild();

130:         revert MustImplementInChild();

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L123

### [N&#x2011;02]  Non-exploitable re-entrancies
Code should follow the best-practice of [check-effects-interaction](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/)

*There are 2 instances of this issue:*

```solidity
File: /contracts/QuestFactory.sol

100              newQuest.transferOwnership(msg.sender);
101:             ++questIdCount;

131              newQuest.transferOwnership(msg.sender);
132:             ++questIdCount;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L100-L101

### [N&#x2011;03]  `Erc1155Quest` doesn't have an option for a protocol fee
It's inconsistent for `Erc20Quest` to support fees, but for `Erc1155Quest` not to

*There is 1 instance of this issue:*

```solidity
File: /contracts/Erc1155Quest.sol

11:  contract Erc1155Quest is Quest, ERC1155Holder {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L11

### [N&#x2011;04]  `_disableInitializers()` is preferred for constructors
Use `_disableInitializers()` in the constructor, rather than using the `initializer` modifier

*There is 1 instance of this issue:*

```solidity
File: /contracts/QuestFactory.sol

35:      constructor() initializer {}

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L35

### [N&#x2011;05]  Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions
See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

*There are 3 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

16:   contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L16

```solidity
File: contracts/RabbitHoleReceipt.sol

15    contract RabbitHoleReceipt is
16        Initializable,
17        ERC721Upgradeable,
18        ERC721EnumerableUpgradeable,
19        ERC721URIStorageUpgradeable,
20        OwnableUpgradeable,
21:       IERC2981Upgradeable

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L15-L21

```solidity
File: contracts/RabbitHoleTickets.sol

11    contract RabbitHoleTickets is
12        Initializable,
13        ERC1155Upgradeable,
14        OwnableUpgradeable,
15        ERC1155BurnableUpgradeable,
16:       IERC2981Upgradeable

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L11-L16

### [N&#x2011;06]  Import declarations should import specific identifiers, rather than the whole file
Using import declarations of the form `import {<identifier_name>} from "some/file.sol"` avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation

*There are 23 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

4:    import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';

10:   import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol';

11:   import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L4

```solidity
File: contracts/RabbitHoleReceipt.sol

4:    import '@openzeppelin/contracts-upgradeable/token/ERC721/ERC721Upgradeable.sol';

5:    import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721URIStorageUpgradeable.sol';

6:    import '@openzeppelin/contracts-upgradeable/token/ERC721/extensions/ERC721EnumerableUpgradeable.sol';

7:    import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';

8:    import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';

9:    import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';

10:   import '@openzeppelin/contracts-upgradeable/utils/CountersUpgradeable.sol';

11:   import './ReceiptRenderer.sol';

12:   import './interfaces/IQuestFactory.sol';

13:   import './interfaces/IQuest.sol';

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L4

```solidity
File: contracts/RabbitHoleTickets.sol

4:    import '@openzeppelin/contracts-upgradeable/token/ERC1155/ERC1155Upgradeable.sol';

5:    import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';

6:    import '@openzeppelin/contracts-upgradeable/token/ERC1155/extensions/ERC1155BurnableUpgradeable.sol';

7:    import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';

8:    import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';

9:    import './TicketRenderer.sol';

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L4

```solidity
File: contracts/ReceiptRenderer.sol

4:    import '@openzeppelin/contracts/utils/Base64.sol';

5:    import '@openzeppelin/contracts/utils/Strings.sol';

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L4

```solidity
File: contracts/TicketRenderer.sol

4:    import '@openzeppelin/contracts/utils/Base64.sol';

5:    import '@openzeppelin/contracts/utils/Strings.sol';

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L4

### [N&#x2011;07]  `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings

*There is 1 instance of this issue:*

```solidity
File: contracts/RabbitHoleTickets.sol

110:          uint256 tokenId_,

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L110

### [N&#x2011;08]  `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*There are 7 instances of this issue:*

```solidity
File: contracts/Erc20Quest.sol

/// @audit 10_000
53:           return (maxTotalRewards() * questFee) / 10_000;

/// @audit 10_000
97:           return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L53

```solidity
File: contracts/QuestFactory.sol

/// @audit 2_000
48:           setQuestFee(2_000);

/// @audit 10_000
187:          if (questFee_ > 10_000) revert QuestFeeTooHigh();

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L48

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit 10_000
184:          uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L184

```solidity
File: contracts/RabbitHoleTickets.sol

/// @audit 10_000
113:          uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L113

```solidity
File: contracts/ReceiptRenderer.sol

/// @audit 20
60:               generateAttribute('Reward Address', Strings.toHexString(uint160(rewardAddress_), 20)),

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L60

### [N&#x2011;09]  Missing event and or timelock for critical parameter change
Events help non-contract tools to track changes, and events prevent users from being surprised by changes

*There are 5 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

159       function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
160           claimSignerAddress = claimSignerAddress_;
161:      }

165       function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
166           if (protocolFeeRecipient_ == address(0)) revert AddressZeroNotAllowed();
167           protocolFeeRecipient = protocolFeeRecipient_;
168:      }

186       function setQuestFee(uint256 questFee_) public onlyOwner {
187           if (questFee_ > 10_000) revert QuestFeeTooHigh();
188           questFee = questFee_;
189:      }

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159-L161

```solidity
File: contracts/RabbitHoleReceipt.sol

71        function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
72            royaltyRecipient = royaltyRecipient_;
73:       }

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71-L73

```solidity
File: contracts/RabbitHoleTickets.sol

60        function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
61            royaltyRecipient = royaltyRecipient_;
62:       }

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L60-L62

### [N&#x2011;10]  Events that mark critical parameter changes should contain both the old and the new value
This should especially be done if the new value is not required to be different from the old value

*There are 4 instances of this issue:*

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit setMinterAddress()
85:           emit MinterAddressSet(minterAddress_);

/// @audit setRoyaltyFee()
92:           emit RoyaltyFeeSet(royaltyFee_);

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L85

```solidity
File: contracts/RabbitHoleTickets.sol

/// @audit setRoyaltyFee()
68:           emit RoyaltyFeeSet(royaltyFee_);

/// @audit setMinterAddress()
75:           emit MinterAddressSet(minterAddress_);

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L68

### [N&#x2011;11]  Expressions for constant values such as a call to `keccak256()`, should use `immutable` rather than `constant`
While it **doesn't save any gas** because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand. There is a difference between `constant` variables and `immutable` variables, and they should each be used in their appropriate contexts. `constants` should be used for literal values written into the code, and `immutable` variables should be used for expressions, or values calculated in, or passed into the constructor.

*There is 1 instance of this issue:*

```solidity
File: contracts/QuestFactory.sol

17:       bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L17

### [N&#x2011;12]  Lines are too long
Usually lines in source code are limited to [80](https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width) characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over [164](https://github.com/aizatto/character-length) characters, the lines below should be split when they reach that length

*There are 3 instances of this issue:*

```solidity
File: contracts/Erc20Quest.sol

56:       /// @notice Starts the quest by marking it ready to start at the contract level. Marking a quest ready to start does not mean that it is live. It also requires that the start time has passed

57:       /// @dev Requires that the balance of the rewards in the contract is greater than or equal to the maximum amount of rewards that can be claimed by all users and the protocol

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L56

```solidity
File: contracts/interfaces/IQuestFactory.sol

16:       event QuestCreated(address indexed creator, address indexed contractAddress, string indexed questId, string contractType, address rewardTokenAddress, uint256 endTime, uint256 startTime, uint256 totalParticipants, uint256 rewardAmountOrTokenId);

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L16

### [N&#x2011;13]  Non-library/interface files should use fixed compiler versions, not floating ones

*There are 8 instances of this issue:*

```solidity
File: contracts/Erc1155Quest.sol

2:    pragma solidity ^0.8.15;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L2

```solidity
File: contracts/Erc20Quest.sol

2:    pragma solidity ^0.8.15;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L2

```solidity
File: contracts/QuestFactory.sol

2:    pragma solidity ^0.8.15;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L2

```solidity
File: contracts/Quest.sol

2:    pragma solidity ^0.8.15;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L2

```solidity
File: contracts/RabbitHoleReceipt.sol

2:    pragma solidity ^0.8.15;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L2

```solidity
File: contracts/RabbitHoleTickets.sol

2:    pragma solidity ^0.8.15;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L2

```solidity
File: contracts/ReceiptRenderer.sol

2:    pragma solidity ^0.8.15;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L2

```solidity
File: contracts/TicketRenderer.sol

2:    pragma solidity ^0.8.15;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L2

### [N&#x2011;14]  Typos

*There is 1 instance of this issue:*

```solidity
File: contracts/QuestFactory.sol

/// @audit remave
176:      /// @dev set or remave a contract address to be used as a reward

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L176

### [N&#x2011;15]  File is missing NatSpec

*There are 2 instances of this issue:*

```solidity
File: contracts/interfaces/IQuestFactory.sol


```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol

```solidity
File: contracts/interfaces/IQuest.sol


```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol

### [N&#x2011;16]  NatSpec is incomplete

*There are 10 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

/// @audit Missing: '@return'
191       /// @dev return the number of minted receipts for a quest
192       /// @param questId_ The id of the quest
193:      function getNumberMinted(string memory questId_) external view returns (uint) {

/// @audit Missing: '@return'
197       /// @dev return data in the quest struct for a questId
198       /// @param questId_ The id of the quest
199:      function questInfo(string memory questId_) external view returns (address, uint, uint) {

/// @audit Missing: '@return'
208       /// @param hash_ The hash of the message
209       /// @param signature_ The signature of the hash
210:      function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L191-L193

```solidity
File: contracts/Quest.sol

/// @audit Missing: '@return'
133       /// @notice Checks if a Receipt token id has been used to claim a reward
134       /// @param tokenId_ The token id to check
135:      function isClaimed(uint256 tokenId_) public view returns (bool) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L133-L135

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit Missing: '@return'
107       /// @param questId_ the quest id
108       /// @param claimingAddress_ the address claiming to own the tokens
109       function getOwnedTokenIdsOfQuest(
110           string memory questId_,
111           address claimingAddress_
112:      ) public view returns (uint[] memory) {

/// @audit Missing: '@return'
176       /// @param tokenId_ the token id
177       /// @param salePrice_ the sale price
178       function royaltyInfo(
179           uint256 tokenId_,
180           uint256 salePrice_
181:      ) external view override returns (address receiver, uint256 royaltyAmount) {

/// @audit Missing: '@return'
188       /// @dev returns true if the supplied interface id is supported
189       /// @param interfaceId_ the interface id
190       function supportsInterface(
191           bytes4 interfaceId_
192:      ) public view virtual override(ERC721Upgradeable, ERC721EnumerableUpgradeable, IERC165Upgradeable) returns (bool) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L107-L112

```solidity
File: contracts/RabbitHoleTickets.sol

/// @audit Missing: '@return'
107       /// @param tokenId_ the token id
108       /// @param salePrice_ the sale price
109       function royaltyInfo(
110           uint256 tokenId_,
111           uint256 salePrice_
112:      ) external view override returns (address receiver, uint256 royaltyAmount) {

/// @audit Missing: '@return'
117       /// @dev returns true if the supplied interface id is supported
118       /// @param interfaceId_ the interface id
119       function supportsInterface(
120           bytes4 interfaceId_
121:      ) public view virtual override(ERC1155Upgradeable, IERC165Upgradeable) returns (bool) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L107-L112

```solidity
File: contracts/ReceiptRenderer.sol

/// @audit Missing: '@return'
80        /// @param key The key for the attribute
81        /// @param value The value for the attribute
82:       function generateAttribute(string memory key, string memory value) public pure returns (string memory) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L80-L82

### [N&#x2011;17]  Not using the named return variables anywhere in the function is confusing
Consider changing the variable to be an unnamed one

*There are 4 instances of this issue:*

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit receiver
/// @audit royaltyAmount
178       function royaltyInfo(
179           uint256 tokenId_,
180           uint256 salePrice_
181:      ) external view override returns (address receiver, uint256 royaltyAmount) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L178-L181

```solidity
File: contracts/RabbitHoleTickets.sol

/// @audit receiver
/// @audit royaltyAmount
109       function royaltyInfo(
110           uint256 tokenId_,
111           uint256 salePrice_
112:      ) external view override returns (address receiver, uint256 royaltyAmount) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L109-L112

### [N&#x2011;18]  Contracts should have full test coverage
While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

*There is 1 instance of this issue:*

```solidity
File: Various Files


```

### [N&#x2011;19]  Large or complicated code bases should implement fuzzing tests
Large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts, should implement [fuzzing tests](https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05). Fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.

*There is 1 instance of this issue:*

```solidity
File: Various Files


```

### [N&#x2011;20]  Function ordering does not follow the Solidity style guide
According to the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), functions should be laid out in the following order :`constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal`, `private`, but the cases below do not follow this pattern

*There are 9 instances of this issue:*

```solidity
File: contracts/Erc1155Quest.sol

/// @audit _calculateRewards() came earlier
54:       function withdrawRemainingTokens(address to_) public override onlyOwner {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54

```solidity
File: contracts/Erc20Quest.sol

/// @audit _calculateRewards() came earlier
81:       function withdrawRemainingTokens(address to_) public override onlyOwner {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81

```solidity
File: contracts/QuestFactory.sol

/// @audit grantDefaultAdminAndCreateQuestRole() came earlier
159:      function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {

/// @audit setQuestFee() came earlier
193:      function getNumberMinted(string memory questId_) external view returns (uint) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159

```solidity
File: contracts/Quest.sol

/// @audit _setClaimed() came earlier
96:       function claim() public virtual onlyQuestActive {

/// @audit _transferRewards() came earlier
135:      function isClaimed(uint256 tokenId_) public view returns (bool) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit _beforeTokenTransfer() came earlier
158       function tokenURI(
159           uint tokenId_
160:      ) public view virtual override(ERC721Upgradeable, ERC721URIStorageUpgradeable) returns (string memory) {

/// @audit tokenURI() came earlier
178       function royaltyInfo(
179           uint256 tokenId_,
180           uint256 salePrice_
181:      ) external view override returns (address receiver, uint256 royaltyAmount) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L158-L160

```solidity
File: contracts/RabbitHoleTickets.sol

/// @audit uri() came earlier
109       function royaltyInfo(
110           uint256 tokenId_,
111           uint256 salePrice_
112:      ) external view override returns (address receiver, uint256 royaltyAmount) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L109-L112

### [N&#x2011;21]  Contract does not follow the Solidity style guide's suggested layout ordering
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be 1) Type declarations, 2) State variables, 3) Events, 4) Modifiers, and 5) Functions, but the contract(s) below do not follow this ordering

*There are 5 instances of this issue:*

```solidity
File: contracts/Quest.sol

/// @audit function _setClaimed came earlier
76        modifier onlyAdminWithdrawAfterEnd() {
77            if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
78            _;
79:       }

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76-L79

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit event MinterAddressSet came earlier
27:       CountersUpgradeable.Counter private _tokenIds;

/// @audit function initialize came earlier
58        modifier onlyMinter() {
59            msg.sender == minterAddress;
60            _;
61:       }

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L27

```solidity
File: contracts/RabbitHoleTickets.sol

/// @audit event MinterAddressSet came earlier
22:       address public royaltyRecipient;

/// @audit function initialize came earlier
47        modifier onlyMinter() {
48            msg.sender == minterAddress;
49            _;
50:       }

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L22


___

## Excluded findings
These findings are excluded from awards calculations because there are publicly-available automated tools that find them. The valid ones appear here for completeness

## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Missing checks for `address(0x0)` when assigning values to `address` state variables | 12 | 
| [L&#x2011;02] | `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 6 | 

Total: 18 instances over 2 issues


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | `public` functions not called by the contract should be declared `external` instead | 27 | 
| [N&#x2011;02] | Event is missing `indexed` fields | 1 | 

Total: 28 instances over 2 issues





## Low Risk Issues

### [L&#x2011;01]  Missing checks for `address(0x0)` when assigning values to `address` state variables

*There are 12 instances of this issue:*

```solidity
File: contracts/Erc20Quest.sol

/// @audit (valid but excluded finding)
39:           protocolFeeRecipient = protocolFeeRecipient_;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L39

```solidity
File: contracts/QuestFactory.sol

/// @audit (valid but excluded finding)
45:           claimSignerAddress = claimSignerAddress_;

/// @audit (valid but excluded finding)
160:          claimSignerAddress = claimSignerAddress_;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L45

```solidity
File: contracts/Quest.sol

/// @audit (valid but excluded finding)
40:           rewardToken = rewardTokenAddress_;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L40

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit (valid but excluded finding)
52:           royaltyRecipient = royaltyRecipient_;

/// @audit (valid but excluded finding)
53:           minterAddress = minterAddress_;

/// @audit (valid but excluded finding)
72:           royaltyRecipient = royaltyRecipient_;

/// @audit (valid but excluded finding)
84:           minterAddress = minterAddress_;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L52

```solidity
File: contracts/RabbitHoleTickets.sol

/// @audit (valid but excluded finding)
41:           royaltyRecipient = royaltyRecipient_;

/// @audit (valid but excluded finding)
42:           minterAddress = minterAddress_;

/// @audit (valid but excluded finding)
61:           royaltyRecipient = royaltyRecipient_;

/// @audit (valid but excluded finding)
74:           minterAddress = minterAddress_;

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L41

### [L&#x2011;02]  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).

If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*There are 6 instances of this issue:*

```solidity
File: contracts/QuestFactory.sol

/// @audit (valid but excluded finding)
/// @audit (valid but excluded finding)
72:           if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {

/// @audit (valid but excluded finding)
/// @audit (valid but excluded finding)
105:          if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {

/// @audit (valid but excluded finding)
211:          bytes32 messageDigest = keccak256(abi.encodePacked('\x19Ethereum Signed Message:\n32', hash_));

/// @audit (valid but excluded finding)
222:          if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72

## Non-critical Issues

### [N&#x2011;01]  `public` functions not called by the contract should be declared `external` instead
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`.

*There are 27 instances of this issue:*

```solidity
File: contracts/Erc20Quest.sol

/// @audit (valid but excluded finding)
102:      function withdrawFee() public onlyAdminWithdrawAfterEnd {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L102

```solidity
File: contracts/QuestFactory.sol

/// @audit (valid but excluded finding)
61        function createQuest(
62            address rewardTokenAddress_,
63            uint256 endTime_,
64            uint256 startTime_,
65            uint256 totalParticipants_,
66            uint256 rewardAmountOrTokenId_,
67            string memory contractType_,
68            string memory questId_
69:       ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {

/// @audit (valid but excluded finding)
142:      function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {

/// @audit (valid but excluded finding)
159:      function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {

/// @audit (valid but excluded finding)
172:      function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {

/// @audit (valid but excluded finding)
179:      function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {

/// @audit (valid but excluded finding)
219:      function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61-L69

```solidity
File: contracts/Quest.sol

/// @audit (valid but excluded finding)
57:       function pause() public onlyOwner onlyStarted {

/// @audit (valid but excluded finding)
63:       function unPause() public onlyOwner onlyStarted {

/// @audit (valid but excluded finding)
140:      function getRewardAmount() public view returns (uint256) {

/// @audit (valid but excluded finding)
145:      function getRewardToken() public view returns (address) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57

```solidity
File: contracts/RabbitHoleReceipt.sol

/// @audit (valid but excluded finding)
43        function initialize(
44            address receiptRenderer_,
45            address royaltyRecipient_,
46            address minterAddress_,
47            uint royaltyFee_
48:       ) public initializer {

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

/// @audit (valid but excluded finding)
109       function getOwnedTokenIdsOfQuest(
110           string memory questId_,
111           address claimingAddress_
112:      ) public view returns (uint[] memory) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L43-L48

```solidity
File: contracts/RabbitHoleTickets.sol

/// @audit (valid but excluded finding)
32        function initialize(
33            address ticketRenderer_,
34            address royaltyRecipient_,
35            address minterAddress_,
36            uint royaltyFee_
37:       ) public initializer {

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

/// @audit (valid but excluded finding)
92        function mintBatch(
93            address to_,
94            uint256[] memory ids_,
95            uint256[] memory amounts_,
96            bytes memory data_
97:       ) public onlyMinter {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L32-L37

```solidity
File: contracts/TicketRenderer.sol

/// @audit (valid but excluded finding)
16        function generateTokenURI(
17            uint tokenId_
18:       ) public pure returns (string memory) {

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L16-L18

### [N&#x2011;02]  Event is missing `indexed` fields
Index event fields make the field more quickly accessible [to off-chain tools](https://ethereum.stackexchange.com/questions/40396/can-somebody-please-explain-the-concept-of-event-indexing) that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*There is 1 instance of this issue:*

```solidity
File: contracts/interfaces/IQuest.sol

/// @audit (valid but excluded finding)
8:        event Claimed(address account, uint256 amount);

```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L8


