# QA Report for RabbitHole Quest Protocol contest
## Overview
During the audit, 8 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
L-1 | Misleading modifier | Low | 1
L-2 | Owner can renounce ownership | Low | 1
L-3 | Event should be emitted | Low | 19
L-4 | Quest startTime is not checked | Low | 1
L-5 |ECDSAUpgradeable.recover return value is not checked | Low | 2
L-6 | Max limit can be set | Low | 1
L-7 | Sign change | Low | 1
L-8 | Checks can be added | Low | 3

## Low Risk Findings(8)
### L-1. Misleading modifier
##### Description
The modifier ```onlyAdminWithdrawAfterEnd()``` does not check that only admin can call a fucntion, actually anyone can call it.

##### Instances
- [Link](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76-L79):
```
    modifier onlyAdminWithdrawAfterEnd() {
        if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
        _;
    }
```

##### Recommendation
Add a check in modifier or rename it.

#
### L-2. Owner can renounce ownership
##### Description
Openzeppelin's Ownable.sol implements ```renounceOwnership()``` function which leaves the contract without an owner and removes any functionality that is only available to the them.

##### Instances
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L9 

##### Recommendation
Consider reimplementing the function to disable it.
#
### L-3. Event should be emitted
##### Description
The governor functions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes and allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.
##### Instances
- [```function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L142) 
- [```function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159) 
- [```function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L165) 
- [```function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L172) 
- [```function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L179) 
- [```function setQuestFee(uint256 questFee_) public onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L186) 
- [```function setReceiptRenderer(address receiptRenderer_) public onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L65) 
- [```function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71) 
- [```function setQuestFactory(address questFactory_) public onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L77) 
- [```function mint(address to_, string memory questId_) public onlyMinter {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L98) 
- [```function start() public virtual onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50) 
- [```function pause() public onlyOwner onlyStarted {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57) 
- [```function unPause() public onlyOwner onlyStarted {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L63) 
- [```function setTicketRenderer(address ticketRenderer_) public onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L54) 
- [```function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L60) 
- [```function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L83) 
- [```function mintBatch(```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L92) 
- [```function start() public override {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L58) 
- [```function start() public override {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L33) 

##### Recommendation
Consider adding events for these functions.
#
### L-4. Quest startTime is not checked
##### Description
The function ```mintReceipt()``` does not check that quest has started.
##### Instances
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219-L229

##### Recommendation
Consider adding the check.
#
### L-5. ECDSAUpgradeable.recover return value is not checked
##### Description
ECDSAUpgradeable.recover may return address(0) if invoked with an invalid signature. So, it is possible to [```mintReceipt()```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L223) with invalid signature, if ```claimSignerAddress``` accidentally will be set to address(0). ```claimSignerAddress``` can be equal to address(0) if it is stay unset in initialize() or incorrectly set in setClaimSignerAddress() function.
 
##### Instances
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L212
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L223

##### Recommendation
ECDSAUpgradeable.recover return value must be checked in the function recoverSigner() to ensure that it is not zero address.
#
### L-6. Max limit can be set
##### Description
```royaltyFee``` is unlimited.
##### Instances
- (https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L67

##### Recommendation
It is better to limit the maximum ```royaltyFee``` to increase transparency.
#
### L-7. Sign change
##### Description
It is possible that some users will participate in the quest at the last second, so the ```<``` sign needs to be change to ```<=```.
##### Instances
- [```if (block.timestamp < endTime) revert NoWithdrawDuringClaim();```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L77) 

##### Recommendation
Consider changing to:
``` if (block.timestamp <= endTime) revert NoWithdrawDuringClaim();```.
#
### L-8. Checks can be added
##### Description
In Quest constructor it can be checked that:
- ```uint256 totalParticipants_```> 0, 
- ```endTime_ - startTime_ ``` > minimum duration,
- ```endTime_ - startTime_ ``` < maximum duration.

##### Instances
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L26-L46

##### Recommendation
Consider adding more check.
