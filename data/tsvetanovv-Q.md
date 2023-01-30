### QA Issues List
| Number |Issues Details
|:--:|:-------|:--:|
|[QA-01]| Use latest Solidity version
|[QA-02]| Use stable pragma statement
|[QA-03]| Update external dependency to latest version
|[QA-04]|In all solidity files, license keyword should be mentioned
|[QA-05]|Solidity compiler optimizations can be problematic
|[QA-06]| Use named imports instead of plain `IMPORT FILE.SOL`
|[QA-07]|Constant values such as a call to `keccak256()`, should used to immutable rather than constant
|[QA-08]| Missing zero address validation
|[QA-09]| Missing EVENT for critical parameter change
|[QA-10]| Missing uint256 `0` check in `royaltyInfo()` function
|[QA-11]| Open `TODO` in the code
|[QA-12]| Constants should be defined rather than using magic numbers
|[QA-13]| Critical address changes should use two-step procedure
|[QA-14]| Loss of precision due to rounding
|[QA-15]| Omissions in Events
|[QA-16]| Modifier with a confusing name
***

## [QA-01] Use latest Solidity version

Solidity pragma versioning should be upgraded to latest available version. 
***

## [QA-02] Use stable pragma statement

Using a floating pragma statement `pragma solidity ^0.8.15;` is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode.
***

## [QA-03] Update external dependency to latest version

The leatest version is 4.8.1.

```
package.json:

"@openzeppelin/contracts": "4.8.0", 
"@openzeppelin/contracts-upgradeable": "4.8.0",
```
***

## [QA-04] In all solidity files, license keyword should be mentioned
In the following contracts this problem occurs:
```
QuestFactory.sol
Quest.sol
Erc20Quest.sol
Erc1155Quest.sol
IQuest.sol
IQuestFactory.sol
```
***

## [QA-05] Solidity compiler optimizations can be problematic
```
hardhat.config.ts:

compilers: [
      {
        version: '0.8.15',
        settings: {
          optimizer: { //@audit-ok - optimizer
            enabled: true,
            runs: 5000,
          },
        },
      },
    ],
 ```
 
Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.
***

## [QA-06] USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT FILE.SOL`

In the following contracts this problem occurs:
```
QuestFactory.sol
RabbitHoleReceipt.sol
RabbitHoleTickets.sol
ReceiptRenderer.sol
TicketRenderer.sol
```
**Recommendation:**
`import {contract1 , contract2} from "filename.sol";`
***

## [QA-07] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USED TO IMMUTABLE RATHER THAN CONSTANT
There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.
```
QuestFactory.sol:
17: bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
```
***

## [QA-08] Missing zero address validation

Setters of address type parameters should include a zero-address check otherwise contract functionality may become inaccessible or tokens burnt forever.

```
QuestFactory.sol:
142: function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
159: function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
172: function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
179: function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {

RabbitHoleReceipt.sol:
65: function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
71: function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
77: function setQuestFactory(address questFactory_) public onlyOwner {
83: function setMinterAddress(address minterAddress_) public onlyOwner {
98: function mint(address to_, string memory questId_) public onlyMinter {
109: function getOwnedTokenIdsOfQuest(

RabbitHoleTickets.sol:
54: function setTicketRenderer(address ticketRenderer_) public onlyOwner {
60: function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
73: function setMinterAddress(address minterAddress_) public onlyOwner {

Erc20Quest.sol:
81: function withdrawRemainingTokens(address to_) public override onlyOwner {
```

#### Recommendation

Check that the address is not zero.
***

## [QA-09] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.
```
QuestFactory.sol:
159: function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
165: function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
172: function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
179: function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
186: function setQuestFee(uint256 questFee_) public onlyOwner {

RabbitHoleReceipt.sol:
65: function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
71: function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
77: function setQuestFactory(address questFactory_) public onlyOwner {
98: function mint(address to_, string memory questId_) public onlyMinter {

Quest.sol:
50: function start() public virtual onlyOwner {
57: function pause() public onlyOwner onlyStarted {
63: function unPause() public onlyOwner onlyStarted {

RabbitHoleTickets.sol:
54: function setTicketRenderer(address ticketRenderer_) public onlyOwner {
60: function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
```
***

## [QA-10] Missing uint256 `0` check in `royaltyInfo()` function
In `RabbitHoleReceipt.sol` contract, `royaltyInfo()` function missing `0` check.
If `uint256 salePrice_` is zero it is possible to get division by zero error in `uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;`

`RabbitHoleTickets.sol` has the same problem:
```
109: function royaltyInfo(
        uint256 tokenId_,
        uint256 salePrice_
    )
```
***

## [QA-11] Open `TODO` in the code
The code that contains “open todos” reflects that the development is not finished and that the code can change a posteriori, prior release, with or without audit.

`IQuest.sol:`
```
4: // TODO clean this whole thing up
```
***

## [QA-12] Constants should be defined rather than using magic numbers
```
Erc20Quest.sol
53: return (maxTotalRewards() * questFee) / 10_000;
97: return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;

QuestFactory.sol
187: if (questFee_ > 10_000) revert QuestFeeTooHigh();

RabbitHoleTickets.sol
113: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;

RabbitHoleReceipt.sol
184|: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```
***

## [QA-13] CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE
The critical procedures should be a two-step process.
```
QuestFactory.sol:
159: function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
165: function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
172: function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
179: function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {

RabbitHoleReceipt.sol:
65: function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
71: function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
77: function setQuestFactory(address questFactory_) public onlyOwner {
83: function setMinterAddress(address minterAddress_) public onlyOwner {

RabbitHoleTickets.sol:
54: function setTicketRenderer(address ticketRenderer_) public onlyOwner {
60: function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
66: function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
73: function setMinterAddress(address minterAddress_) public onlyOwner {
```

**Recommended Mitigation Steps**
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding a two- step procedure on the critical functions.
***

## [QA-14] LOSS OF PRECISION DUE TO ROUNDING

```
RabbitHoleReceipt.sol
184: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;

RabbitHoleTickets.sol
113: uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;

Erc20Quest.sol:
53: return (maxTotalRewards() * questFee) / 10_000;
```
***

## [QA-15]  Omissions in Events
Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

Events with no old value;
```
RabbitHoleReceipt.sol
83: function setMinterAddress(address minterAddress_) public onlyOwner {
        minterAddress = minterAddress_;
        emit MinterAddressSet(minterAddress_);
    }
	
90: function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
	
RabbitHoleTickets.sol
66: function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }

73: function setMinterAddress(address minterAddress_) public onlyOwner {
        minterAddress = minterAddress_;
        emit MinterAddressSet(minterAddress_);
    }
```
***

## [QA-16] Modifier with a confusing name

In `Quest.sol` we have `onlyAdminWithdrawAfterEnd()` but the name doesn't match what the modifier does. `onlyAdminWithdrawAfterEnd()` no check for admin. This misleading modifier name could lead to big problems in the future. 
Also in `Erc20Quest.sol`  function `withdrawFee()`  uses this modifier which misleads that only an admin can call her.

#### Proof of Concept
Quest.sol: 

```
/// @notice Prevents reward withdrawal until the Quest has ended
76:    modifier onlyAdminWithdrawAfterEnd() {
         if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
         _;
     }
```

Erc20Quest.sol:
```
102: function withdrawFee() public onlyAdminWithdrawAfterEnd {
        IERC20(rewardToken).safeTransfer(protocolFeeRecipient, protocolFee());
    }
```

#### Recommended Mitigation Steps

Correct the name to what modifier does. 