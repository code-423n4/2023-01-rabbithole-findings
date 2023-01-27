## Non Critical Issues
### [NC-1] Avoid repeated logic
Repeated code/logic makes maintainability harder, increases the total deployment size, and makes the code harder to read. Consider moving some of the repeated logic to the enclosing scope.

```solidity
File: QuestFactory.sol#L72-#L134

	if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
		if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

		Erc20Quest newQuest = new Erc20Quest(
			rewardTokenAddress_,
			endTime_,
			startTime_,
			totalParticipants_,
			rewardAmountOrTokenId_,
			questId_,
			address(rabbitholeReceiptContract),
			questFee,
			protocolFeeRecipient
		);

		emit QuestCreated(
			msg.sender,
			address(newQuest),
			questId_,
			contractType_,
			rewardTokenAddress_,
			endTime_,
			startTime_,
			totalParticipants_,
			rewardAmountOrTokenId_
		);
		quests[questId_].questAddress = address(newQuest);
		quests[questId_].totalParticipants = totalParticipants_;
		newQuest.transferOwnership(msg.sender);
		++questIdCount;
		return address(newQuest);
	}

	if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
		if (msg.sender != owner()) revert OnlyOwnerCanCreate1155Quest();

		Erc1155Quest newQuest = new Erc1155Quest(
			rewardTokenAddress_,
			endTime_,
			startTime_,
			totalParticipants_,
			rewardAmountOrTokenId_,
			questId_,
			address(rabbitholeReceiptContract)
		);

		emit QuestCreated(
			msg.sender,
			address(newQuest),
			questId_,
			contractType_,
			rewardTokenAddress_,
			endTime_,
			startTime_,
			totalParticipants_,
			rewardAmountOrTokenId_
		);
		quests[questId_].questAddress = address(newQuest);
		quests[questId_].totalParticipants = totalParticipants_;
		newQuest.transferOwnership(msg.sender);
		++questIdCount;
		return address(newQuest);
	}
```

### [NC-2] State/configuration changes without relevant events
When changing state/configuration, it is best to emit events that accurately reflect the changes.

<i>16 Instances</i>
```solidity
File: Quest.sol

51:		isPaused = false;

52:		hasStarted = true;

58:		isPaused = true;

64:		isPaused = false;

71: 	claimedList[tokenIds_[i]] = true;
  
115: 	redeemedTokens += redeemableTokenCount;
```

```solidity
File: QuestFactory.sol

160: 	claimSignerAddress = claimSignerAddress_;

167:    protocolFeeRecipient = protocolFeeRecipient_;

173:    rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);

180: 	rewardAllowlist[rewardAddress_] = allowed_;

188: 	questFee = questFee_;
```

```solidity
File: RabbitHoleReceipt.sol

66: 	ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);

72: 	royaltyRecipient = royaltyRecipient_;

78: 	QuestFactoryContract = IQuestFactory(questFactory_);
```

```solidity
File: RabbitHoleTickets.sol

55: 	TicketRendererContract = TicketRenderer(ticketRenderer_);

61: 	royaltyRecipient = royaltyRecipient_;
```

### [NC-3] Avoid using non standard versions of common functionality
Common functionality such as pausing contracts, and validating ethereum signed messages have battle tested solutions that should be used in favor of in-house solutions.

```solidity
File: Quest.sol

20:		bool public isPaused;

57:		function pause() public onlyOwner onlyStarted {
58:			isPaused = true;
59:		}

63:		function unPause() public onlyOwner onlyStarted {
64:			isPaused = false;
65:		}
```

This could be replaced with OpenZeppelin's Pauseable contract.

```solidity
File: QuestFactory.sol

211: 	bytes32 messageDigest = keccak256(abi.encodePacked('\x19Ethereum Signed Message:\n32', hash_));
```

This could be replaced with OpenZeppelin's ECDSAUpgradeable.toEthSignedMessageHash function.

### [NC-4] Use OpenZeppelin recommended proxy constructor initialization
According to the [OpenZeppelin guide to writing upgradable contracts](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializing_the_implementation_contract), the `_disableInitializers()` function should be used to disable initialization of the implementation contract.

A fix for this would be to replace:

```solidity
File: QuestFactory.sol

34:		/// @custom:oz-upgrades-unsafe-allow constructor
35:		constructor() initializer {}
```

with this:

```solidity
File: QuestFactory.sol

34:		/// @custom:oz-upgrades-unsafe-allow constructor
35:		constructor() {
36:			_disableInitializers();
37:		}
```

### [NC-5] Message hash unnecessarily passed as parameter
In the `QuestFactory.mintReceipt` a hash and signature are passed as parameters. The hash must be equal to `keccak256(abi.encodePacked(msg.sender, questId_))` or the function will revert. However, since a signer is recovered using the signature and hash, it will revert if an invalid signature-hash pair is passed in. Therefore, it is safe to omit the hash from the parameters, and calculate it in the function.

A fix for this would be to replace:

```solidity
File: QuestFactory.sol

219:	function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {

222:	if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
223:	if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();
```

with this:

```solidity
File: QuestFactory.sol

219:	function mintReceipt(string memory questId_, bytes memory signature_) public {

222:	bytes32 hash_ = keccak256(abi.encodePacked(msg.sender, questId_));
223:	if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();
```

### [NC-6] Variable names should accurately reflect the data they represent
<i>2 Instances</i>

This should be replaced either `rewardAmount`, or `tokenId` depending on whether the contract is an `Erc20Quest` or `Erc1155Quest`. The variable should be stored in the respective contract, instead of the abstract `Quest` contract.
```solidity
File Quest.sol

18:		uint256 public immutable rewardAmountInWeiOrTokenId;
```

This variable reflects the balance of the `claimingAddress_`, not the balance of the `msg.sender`.
```solidity
File RabbitHoleReceipt.sol

113:	uint msgSenderBalance = balanceOf(claimingAddress_);
```

### [NC-7] Boolean literals should not be used for comparisons
Boolean constants can be used directly and do not need to be compare to `true` or `false`.

```slither
File Quest.sol

136:	return claimedList[tokenId_] == true;
```
```slither
File QuestFactory.sol

73:		if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

221:	if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```

### [NC-8] Variables should confirm to Solidity naming conventions
Solidity defines a [naming convention](https://docs.soliditylang.org/en/v0.4.25/style-guide.html#naming-conventions) that should be followed.

```solidity
File: RabbitHoleReceipt.sol

35: 	ReceiptRenderer public ReceiptRendererContract;

36: 	IQuestFactory public QuestFactoryContract;
```
```solidity
File: RabbitHoleTickets.sol

25: 	TicketRenderer public TicketRendererContract;
```


## Low Issues
### [L-1] Add checks on constructor arguments
Since fees are in basis points, fees need to be constrained from 0 to 10,000. 

```solidity
File: RabbitHoleReceipt.sol

54: 	royaltyFee = royaltyFee_;

99: 	royaltyFee = royaltyFee_;
```

```solidity
File: RabbitHoleTickets.sol

43: 	royaltyFee = royaltyFee_;

67: 	royaltyFee = royaltyFee_;
```

```solidity
File: Erc20Quest.sol

38: 	questFee = questFee_;
```

Before setting fees, a check should be made and the function should revert if the new fee is greater than 10,000. For example: ```require(fee <= 10_000, "Fee too high");```.

### [L-2] Quest ID count is incorrect
The `questIdCount` variable is initialized in the constructor to 1, before any quests have been created. This means it is always 1 higher than the actual number of quests.

<i>1 Instance</i>
```solidity
File: QuestFactory.sol

49: 	questIdCount = 1;
```

A fix for this would be to remove this line, which would leave `questIdCount` uninitialized with a value of 0.

### [L-3] Data passed in ERC1155.safeTransferFrom is invalid
In solidity, zero bytes of data can be represented as `""`. While the data `"0x00"` appears to be zero bytes, it is actually UTF-8 encoded as 2 bytes of data.

<i>2 Instances</i>
```solidity
File: Erc1155Quest.sol

42:		IERC1155(rewardToken).safeTransferFrom(address(this), msg.sender, rewardAmountInWeiOrTokenId, amount_, '0x00');


61: 	'0x00'
```

A fix for this would be to replace `'0x00'` with `''`.

### [L-4] Unnecessary combination of access control contracts
In the `QuestFactory` contract, both `OwnableUpgradeable` and `AccessControlUpgradable` are used. It is ambiguous whether admin only functions should only be callable by `owner()`, by those with the `DEFAULT_ADMIN_ROLE`, or some other criteria.

Currently the only role other than the `DEFAULT_ADMIN_ROLE` is the `CREATE_QUEST_ROLE` (#L17). Instead of inheriting `AccessControlUpgradable` for this one role, it would be more size efficient and less ambiguous to have mapping of addresses to whether they can create quests. A possible implementation is as follows:

```solidity
File: QuestFactory.sol

contract QuestFactory /* ----- */ {
	/* ----- */

	mapping(address => bool) public isQuestCreator;

	event ChangeQuestCreator(address indexed account, bool indexed isQuestCreator);

	modifier onlyQuestCreator {
		require(isQuestCreator[msg.sender], "Unauthorized caller");
		_;
	}

	function changeCreateQuestRole(address account, bool isQuestCreator_) public onlyOwner {
		isQuestCreator[account] = isQuestCreator_;
		emit ChangeQuestCreator(account, isQuestCreator_);
	}

	/* ----- */
}
```

### [L-5] Function grows increasingly expensive as users obtain tokens
In the `RabbitHoleReceipt.getOwnedTokenIdsOfQuest`, an iteration is performed over every receipt the user holds (claimed or unclaimed). This could grow to be very expensive as users accrue many receipts.

```solidity
File RabbitHoleReceipt.sol#L112-#L135

function getOwnedTokenIdsOfQuest(
	string memory questId_,
	address claimingAddress_
) public view returns (uint[] memory) {
	uint msgSenderBalance = balanceOf(claimingAddress_);
	uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
	uint foundTokens = 0;

	for (uint i = 0; i < msgSenderBalance; i++) { // iteration over all tokens
		uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
		if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
			tokenIdsForQuest[i] = tokenId;
			foundTokens++;
		}
	}

	uint[] memory filteredTokens = new uint[](foundTokens);
	uint filterTokensIndexTracker = 0;

	for (uint i = 0; i < msgSenderBalance; i++) { // iteration over all unclaimed tokens
		if (tokenIdsForQuest[i] > 0) {
			filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
			filterTokensIndexTracker++;
		}
	}
	return filteredTokens;
}
```

A solution to this is to allow tokens to be claimed by the user directly passing in an array of token ID's that would be calculated off chain. This would remove the requirement for `ERC721EnumerableUpgradeable` to be used by reducing transfer gas cost and contract size.

### [L-6] Function names should accurately reflect their use case
The function `Quest.withdrawRemainingTokens` only performs access control. The function name is misleading. It should be renamed to better reflect its purpose. For example: `Quest.validateWithdrawRemainingTokens`.

```solidity
File: Quest.sol

150:	function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
```