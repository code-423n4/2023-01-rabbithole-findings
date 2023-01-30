# Summary

## Low Risk and Non-Critical Issues

## Non-Critical Issues

|      | Issue                                                      |
| ---- | :--------------------------------------------------------- |
| NC-1 | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)       |
| NC-2 | Omissions in events                                        |
| NC-3 | Use a more recent version of solidity                      |
| NC-4 | For functions, follow solidity standard naming conventions |
| NC-5 | Lines are too long                                         |
| NC-6 | NOT VERIFIED INPUT                                         |

### [NC-1] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)

Rather than using `abi.encodePacked` for appending bytes, since version 0.8.4, `bytes.concat()` is enabled

Since version 0.8.4 for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked(,)`.

#### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/QuestFactory.sol

72:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {

105:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {

211:         bytes32 messageDigest = keccak256(abi.encodePacked('\x19Ethereum Signed Message:\n32', hash_));

222:         if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();

```

```solidity
File: quest-protocol/contracts/ReceiptRenderer.sol

37:         return string(abi.encodePacked('data:application/json;base64,', Base64.encode(dataURI)));

113:         return string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));

```

```solidity
File: quest-protocol/contracts/TicketRenderer.sol

30:         return string(abi.encodePacked('data:application/json;base64,', Base64.encode(dataURI)));

46:         return string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));

```

### [NC-2] Omissions in events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

The events should include the new value and old value where possible.

#### **Proof Of Concept**

Events with no old value:

```solidity
File: quest-protocol/contracts/RabbitHoleReceipt.sol

85:         emit MinterAddressSet(minterAddress_);

92:         emit RoyaltyFeeSet(royaltyFee_);

```

```solidity
File: quest-protocol/contracts/RabbitHoleTickets.sol

68:         emit RoyaltyFeeSet(royaltyFee_);

75:         emit MinterAddressSet(minterAddress_);

```

### [NC-3] Use a more recent version of solidity

#### **Context:**

All Contracts

#### **Description:**

For security, it is best practice to use the latest Solidity version.

For the security fix list in the versions.

https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Recommendation**

Old version of Solidity is used , newer version can be used `(0.8.17)`

### [NC-4] For functions, follow solidity standard naming conventions

Solidity standard naming convention for internal and private functions: the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)

#### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/QuestFactory.sol

152:     function grantDefaultAdminAndCreateQuestRole(address account_) internal {

```

#### **Recommendation**

Use solidity standard naming convention for internal and private functions

### [NC-5] Lines are too long

#### **Context:**

All Contracts apart from quest-protocol/contracts/interfaces/IQuest.sol

#### **Description:**

Usually lines in source code are limited to 80 characters.

[Reference](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length)

#### **Recommendation**

The lines should be split when they reach that length.

### [NC-6] NOT VERIFIED INPUT

External / public functions parameters should be validated to make sure the address is not 0.

Otherwise if not given the right input it can mistakenly lead to loss of user funds.

#### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/Erc1155Quest.sol

54:     function withdrawRemainingTokens(address to_) public override onlyOwner {

```

```solidity
File: quest-protocol/contracts/Erc20Quest.sol

81:     function withdrawRemainingTokens(address to_) public override onlyOwner {

```

```solidity
File: quest-protocol/contracts/Quest.sol

150:     function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}

```

```solidity
File: quest-protocol/contracts/QuestFactory.sol

142:     function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {

159:     function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {

172:     function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {

179:     function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {

```

```solidity
File: quest-protocol/contracts/RabbitHoleReceipt.sol

65:     function setReceiptRenderer(address receiptRenderer_) public onlyOwner {

71:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {

77:     function setQuestFactory(address questFactory_) public onlyOwner {

83:     function setMinterAddress(address minterAddress_) public onlyOwner {

```

```solidity
File: quest-protocol/contracts/RabbitHoleTickets.sol

54:     function setTicketRenderer(address ticketRenderer_) public onlyOwner {

60:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {

67:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {

73:     function setMinterAddress(address minterAddress_) public onlyOwner {

```

## Low Issues

|     | Issue                                                               |
| --- | :------------------------------------------------------------------ |
| L-1 | LOSS OF PRECISION DUE TO ROUNDING                                   |
| L-2 | REQUIRE MESSAGES ARE TOO SHORT AND UNCLEAR                          |
| L-3 | USE `SAFETRANSFEROWNERSHIP` INSTEAD OF `TRANSFEROWNERSHIP` FUNCTION |
| L-4 | Unspecific compiler version pragma                                  |

### [L-1] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/Erc20Quest.sol

53:         return (maxTotalRewards() * questFee) / 10_000;

97:         return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;

```

```solidity
File: quest-protocol/contracts/RabbitHoleReceipt.sol

184:         uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;

```

```solidity
File: quest-protocol/contracts/RabbitHoleTickets.sol

113:         uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;

```

### [L-2] REQUIRE MESSAGES ARE TOO SHORT AND UNCLEAR

The correct and clear error description explains to the user why the function reverts, but the error descriptions below in the project are not self-explanatory. These error descriptions are very important in the debug features of DApps like Tenderly. Error definitions should be added to the require block, not exceeding 32 bytes.

#### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/RabbitHoleReceipt.sol

161:         require(_exists(tokenId_), 'ERC721URIStorage: URI query for nonexistent token');

```

#### Recommended Mitigation Steps:

Error definitions should be added to the require block, not exceeding 32 bytes or we should use custom errors

### [L-3] USE `SAFETRANSFEROWNERSHIP` INSTEAD OF `TRANSFEROWNERSHIP` FUNCTION

`transferOwnership` function is used to change Ownership.

Use a 2 structure `transferOwnership` which is safer.

`safeTransferOwnership`, use it is more secure due to 2-stage ownership transfer.

#### **Proof Of Concept**

```solidity
File: quest-protocol/contracts/QuestFactory.sol

100:             newQuest.transferOwnership(msg.sender);

131:             newQuest.transferOwnership(msg.sender);

```

#### Recommended Mitigation Steps:

Use Ownable2Step.sol

### [L-4] Unspecific compiler version pragma

#### **Context:**

All Contracts
