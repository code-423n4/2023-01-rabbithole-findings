# Report
## Low Risk ##
### [L-1]: Critical changes should use two-step procedure
**Context:**

1. ```function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {``` [L159](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159) 
1. ```function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {``` [L165](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L165) 
1. ```function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {``` [L71](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71) 
1. ```function setMinterAddress(address minterAddress_) public onlyOwner {``` [L83](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L83) 
1. ```function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {``` [L60](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L60) 
1. ```function setMinterAddress(address minterAddress_) public onlyOwner {``` [L73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L73) 

**Recommendation:**

The best practice is to use two-step procedure for critical changes to make them less error-prone. 

### [L-2]: Owner can renounce ownership
**Context:**

```import {OwnableUpgradeable} from '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';``` [L9](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L9) 

**Description:**

@openzeppelin/contracts/access/Ownable.sol' used in this project contract implements renounceOwnership() function. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

**Recommendation:**

You need to reimplement the function.

## Non-Critical Issues ##

### [N-1]: Function defines a named return variable but then it uses return statements
**Context:**

1. ```return (royaltyRecipient, royaltyPayment);``` [L185](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L185) 
1. ```return (royaltyRecipient, royaltyPayment);``` [L114](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L114) 

**Recommendation:**

Choose named return variable or return statement. It is unnecessary to use both.

### [N-2]: Use of immutable instead of constant keccak expression
**Context:**

1. ```bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');``` [L17](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L17) 

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/contracts.html#constant-and-immutable-state-variables) for a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time. It is recommended to use immutable instead. 

### [N-3]: No same value input check
**Context:**

1. ```claimSignerAddress = claimSignerAddress_;``` [L160](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L160) 
1. ```protocolFeeRecipient = protocolFeeRecipient_;``` [L167](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L167) 
1. ```rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);``` [L173](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L173) 
1. ```rewardAllowlist[rewardAddress_] = allowed_;``` [L180](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L180) 
1. ```questFee = questFee_;``` [L188](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L188) 
1. ```ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);``` [L66](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L66) 
1. ```royaltyRecipient = royaltyRecipient_;``` [L72](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L72) 
1. ```QuestFactoryContract = IQuestFactory(questFactory_);``` [L78](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L78) 
1. ```minterAddress = minterAddress_;``` [L84](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L84) 
1. ```royaltyFee = royaltyFee_;``` [L91](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L91) 
1. ```TicketRendererContract = TicketRenderer(ticketRenderer_);``` [L55](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L55) 
1. ```royaltyRecipient = royaltyRecipient_;``` [L61](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L61) 
1. ```royaltyFee = royaltyFee_;``` [L67](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L67) 
1. ```minterAddress = minterAddress_;``` [L74](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L74) 

**Recommendation:**

Example how to fix ```require(_newOwner != owner, " Same address");```

### [N-4]: Wrong order of functions
**Context:**

1. ```using CountersUpgradeable for CountersUpgradeable.Counter;``` [L26](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L26) (using for declaration can not go after event definition)
1. ```modifier onlyAdminWithdrawAfterEnd() {``` [L76](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76) (modifier definition can not go after private function)
1. ```function withdrawRemainingTokens(address to_) public override onlyOwner {``` [L54](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54) (public function can not go after internal function)

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) functions should be grouped according to their visibility and ordered:

+ constructor

+ receive function (if exists)

+ fallback function (if exists)

+ external

+ public

+ internal

+ private

Within a grouping, place the view and pure functions last.

**Recommendation:**

Put the functions in the correct order according to the documentation.

### [N-5]: NatSpec is missing
**Context:**

1. ```function initialize(``` [L37](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37) 
1. ```function initialize(``` [L43](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L43) 
1. ```constructor(``` [L26](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L26) 
1. ```function initialize(``` [L32](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L32) 
1. ```constructor(``` [L17](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L17) 
1. ```constructor(``` [L13](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L13) 
1. ```function generateDataURI(``` [L40](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L40) 

### [N-6]: Typos
**Context:**

1. ```/// @dev set or remave a contract address to be used as a reward``` [L176](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L176) (Change **remave** to **remove**)

### [N-7]: Line is too long
**Context:**

1. ```return ReceiptRendererContract.generateTokenURI(tokenId_, questId, totalParticipants, claimed, rewardAmount, rewardAddress);``` [L172](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L172) 
1. ```/// @dev Only the owner of the Quest can call this function. Also requires that the Quest has started (not by date, but by calling the start function)``` [L56](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L56) 
1. ```/// @dev Only the owner of the Quest can call this function. Also requires that the Quest has started (not by date, but by calling the start function)``` [L62](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L62) 
1. ```/// @dev Function that gets the maximum amount of rewards that can be claimed by all users. It does not include the protocol fee``` [L43](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L43) 
1. ```/// @notice Starts the quest by marking it ready to start at the contract level. Marking a quest ready to start does not mean that it is live. It also requires that the start time has passed``` [L56](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L56) 
1. ```/// @dev Requires that the balance of the rewards in the contract is greater than or equal to the maximum amount of rewards that can be claimed by all users and the protocol``` [L57](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L57) 
1. ```/// @dev Every receipt minted should still be able to claim rewards (and cannot be withdrawn). This function can only be called after the quest end time``` [L79](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L79) 

**Description:**

Maximum suggested line length is [120 characters](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length).
