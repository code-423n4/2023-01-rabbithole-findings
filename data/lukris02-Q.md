# QA Report for RabbitHole Quest Protocol contest
## Overview
During the audit, 8 low and 7 non-critical issues were found.

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
NC-1 | Unused named return variables | Non-Critical | 2
NC-2 | Order of Functions | Non-Critical | 1
NC-3 | Order of Layout | Non-Critical | 3
NC-4 | Missing leading underscores | Non-Critical | 2
NC-5 | Typos | Non-Critical | 1
NC-6 | Open TODOs | Non-Critical | 1
NC-7 | Maximum line length exceeded | Non-Critical | 1

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
#
## Non-Critical Risk Findings(7)
### NC-1. Unused named return variables
##### Description
Both named return variable(s) and return statement are used.
##### Instances
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L185
- https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L114

##### Recommendation
To improve clarity use only named return variables.  
For example, change:
```
function functionName() returns (uint id) {
    return x;
```
to
```
function functionName() returns (uint id) {
    id = x;
```
#
### NC-2. Order of Functions
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-functions), ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.  
Functions should be grouped according to their visibility and ordered:
1) constructor
2) receive function (if exists)
3) fallback function (if exists)
4) external
5) public
6) internal
7) private
##### Instances
- [```function withdrawRemainingTokens(address to_) public override onlyOwner {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54) (public function should be placed before internal)

##### Recommendation
Reorder functions where possible.
#
### NC-3. Order of Layout
##### Description
According to [Order of Layout](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout), inside each contract, library or interface, use the following order:
1) Type declarations
2) State variables
3) Events
4) Modifiers
5) Functions
##### Instances
- [```modifier onlyAdminWithdrawAfterEnd() {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76) 
- [```modifier onlyStarted() {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L82) 
- [```modifier onlyQuestActive() {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L88) 

##### Recommendation
Place modifiers before constructor.
#
### NC-4. Missing leading underscores 
##### Description
Internal and private state variables and functions should have a leading underscore:
##### Instances
- [```function grantDefaultAdminAndCreateQuestRole(address account_) internal {```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152) 
- [```mapping(uint256 => bool) private claimedList;```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L24) 

##### Recommendation
Add leading underscores where needed.
#
### NC-5. Typos
##### Instances
- [```/// @dev set or remave a contract address to be used as a reward```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L176) => ```remove```

#
### NC-6. Open TODOs
##### Instances
- [```// TODO clean this whole thing up```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L4) 

##### Recommendation
Resolve issues.
#
### NC-7. Maximum line length exceeded
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#maximum-line-length), maximum suggested line length is 120 characters.  Longer lines make the code harder to read.
##### Instances
- [```return ReceiptRendererContract.generateTokenURI(tokenId_, questId, totalParticipants, claimed, rewardAmount, rewardAddress);```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L172) 

##### Recommendation
Make the lines shorter.
#