## Low Risk Issues List
Issue
## [L-01] USE SAFETRANSFEROWNERSHIP INSTEAD OF TRANSFEROWNERSHIP FUNCTION
## [L-02] LOSS OF PRECISION DUE TO ROUNDING
## [L-03] ERC1155 has reentrancy possibilities.
Total: 09 contexts over 03 issues

## [L-01] USE SAFETRANSFEROWNERSHIP INSTEAD OF TRANSFEROWNERSHIP FUNCTION
Context: 02
File: QuestFactory.sol
9: import {OwnableUpgradeable} from '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
16: contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgradeable, IQuestFactory {
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L9
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L16
Description:
transferOwnership function is used to change Ownership from OwnableUpgradeable.sol.
Use a 2 structure transferOwnership which is safer.
safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.
Recommendation:
Use Ownable2Step.sol
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol


## [L-02] LOSS OF PRECISION DUE TO ROUNDING
Contexts: 04
File: RabbitHoleReceipt.sol
184: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L184

File: RabbitHoleTickets.sol
113: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L113

File: Erc20Quest.sol
53: return (maxTotalRewards() * questFee) / 10_000;
97: return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L53
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L97

## [L-03] ERC1155 has reentrancy possibilities.
Contexts: 03
Impact
ERC1155 tokens have a callback on transfer, making reentrancy a possibility.
I haven't been able to find any reentrancy, but having extra external function calls isn't safe.
If it's necessary to use an ERC1155 there is nothing you can do about it, but otherwise consider just using an ERC20.
Proof Of Concept
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol
Recommended Mitigation Steps
Confirm that using tokens with callbacks is really necessary for the protocol to function.


## Non-Critical Issues List
Issue:
## [N-01] NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES
## [N-02] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS
## [N-03] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)
## [N-04] It's better to emit after all processing is done
## [N-05] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES
Total: 60 contexts over 05 issues

## [N-01] NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES
Description:
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
https://swcregistry.io/docs/SWC-103
Recommendation:
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/
Contexts: 08
2: pragma solidity ^0.8.15;
01: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol
02: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol
03: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol
04: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol
05: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol
06: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol
07: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol
08: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol

## [N-02] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS
Context: 10
All Contracts
Description:
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
Recommendation:
NatSpec comments should be increased in contracts


## [N-03] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)
Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled
Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).
Contexts: 16
File: QuestFactory.sol
72:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
72:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
105:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
105:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
211:         bytes32 messageDigest = keccak256(abi.encodePacked('\x19Ethereum Signed Message:\n32', hash_));
222:         if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L105
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L211
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L222

File: ReceiptRenderer.sol
37: return string(abi.encodePacked('data:application/json;base64,', Base64.encode(dataURI)));
48: bytes memory attributes = abi.encodePacked(
63: bytes memory dataURI = abi.encodePacked(
83: bytes memory attribute = abi.encodePacked(
101: bytes memory svg = abi.encodePacked(
113: return string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L37
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L48
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L63
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L83
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L101
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L113

File: TicketRenderer.sol
19: bytes memory dataURI = abi.encodePacked(
30: return string(abi.encodePacked('data:application/json;base64,', Base64.encode(dataURI)));
37: bytes memory svg = abi.encodePacked(
46: return string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L19
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L30
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L37
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L46


## [N-04] It's better to emit after all processing is done
Contexts: 03
File: QuestFactory.sol
87: emit QuestCreated(
118: emit QuestCreated(
227: emit ReceiptMinted(msg.sender, questId_);
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L87-L103
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L118-L134
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L227


## [N-05] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES
Context: 23
File: QuestFactory.sol
4: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
10: import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol';
11: import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L4
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L10
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L11

File:  RabbitHoleReceipt.sol
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
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L4-L13

File: RabbitHoleTickets.sol
4: import '@openzeppelin/contracts-upgradeable/token/ERC1155/ERC1155Upgradeable.sol';
5: import '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';
6: import '@openzeppelin/contracts-upgradeable/token/ERC1155/extensions/ERC1155BurnableUpgradeable.sol';
7: import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
8: import '@openzeppelin/contracts-upgradeable/interfaces/IERC2981Upgradeable.sol';
9: import './TicketRenderer.sol';
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L4-L9

File: ReceiptRenderer.sol
4: import '@openzeppelin/contracts/utils/Base64.sol';
5: import '@openzeppelin/contracts/utils/Strings.sol';
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L4-L5

File: TicketRenderer.sol
4: import '@openzeppelin/contracts/utils/Base64.sol';
5: import '@openzeppelin/contracts/utils/Strings.sol';
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L4-L5
Description:
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
Recommendation:
import {contract1 , contract2} from "filename.sol";
