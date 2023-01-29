
# LOW

## There is no validation to the duration of the quest
Since there is no validation on the duration of the quest a user (malicius or by mistake) could set a quest of 1 second;

This is valid (creating a quest of only 1 second);
```solidity
uint256 startDate = block.timestamp + 1 day;
uint256 expiryDate = startDate + 1;

questFactory.createQuest(
  address(sampleERC20),
  expiryDate,
  startDate,
  totalParticipants,
  rewardAmount,
  'erc20',
  'questid'
);
```

Recommendation, add a check for minimal duration on [Quest.sol#L37](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L37) so instead of;
```solidity
        if (endTime_ <= startTime_) revert EndTimeLessThanOrEqualToStartTime();
```
Use
```solidity
        if (endTime_ - startTime_ > MINIMAL_DURATION) revert EndTimeLessThanOrEqualToStartTimeOrBadDuration();
```
If `endTime_ > startTime_` it will fail due underflow.

## Reentrancy issue on `claim()` for `Erc1155Quest`
There is a reentrancy issue when claiming ERC1155 tokens, that will you reenter before `redeemedTokens` is updated.
Here is the callback [Quest.sol#L114](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L114)

Stick to the check effects itteration to avoid this issue or add a reentrancyguard;
```diff
diff --git a/contracts/Quest.sol b/contracts/Quest.sol
index 33163ee..372d2ca 100644
--- a/contracts/Quest.sol
+++ b/contracts/Quest.sol
@@ -111,10 +111,10 @@ contract Quest is Ownable, IQuest {
 
         uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);
         _setClaimed(tokens);
-        _transferRewards(totalRedeemableRewards);
         redeemedTokens += redeemableTokenCount;
 
         emit Claimed(msg.sender, totalRedeemableRewards);
+        _transferRewards(totalRedeemableRewards);
     }
 
     /// @notice Calculate the amount of rewards
```


## Missing upgradability `gap` on upgradeble contracts

Please read [storage_gaps](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps)

Its recommend to add gaps to upgradeable contract to reserve space.

Add this to the state variables of contracts [`QuestFactory`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L16), [`RabbitHoleReceipt`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L15) and [`RabbitHoleTickets`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L11).

```
  // Reserved storage space to allow for layout changes in the future.
  uint256[50] private ______gap;
```

## Current default fee on `QuestFactory` is 20%, seems to high
On [QuestFactory.sol#L48](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L48);
`setQuestFee(2_000);` this translate in a fee of 20% which is pretty high.
Also the owner cant set fee to `100_00` that means that protocol fee could be 100%, that usually mean that its possible to rug.
Its recommend to have a lower maximum.

## Missing `onlyOwner` on `start`
Ok, so `Quest.start` has an `onlyOwner` but the functions override on [Erc20Quest.sol#L58](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L58) and [Erc1155Quest.sol#L33](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L33) doesnt, they do work mainly because you run `super.start()`

Recommendation, add `onlyOwner` modifier on `Erc20Quest.sol#L58` and `Erc1155Quest.sol#L33`

## Anyone can call `withdrawFee`
The function `withdrawFee` on [Erc20Quest.sol#L102](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L102) has a modifier that its name `onlyAdminWithdrawAfterEnd` but it nos doing a `onlyAdmin` check, so anyone can trigger it after the quest end.

## Missing modifier `onlyAdminWithdrawAfterEnd` on `withdrawRemainingTokens`
On [Erc1155Quest.sol#L54](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54) it only has a `onlyOwner` modifier but its missing a `onlyAdminWithdrawAfterEnd` modifier, however this modifier check is done on `super.withdrawRemainingTokens(to_)` but i think its better for readibility to add the modifier here too.

## Missing `address(0)`check on `withdrawRemainingTokens`
Most of the tokens will revert on a transfer to `address(0)` but not all, add a `address(0)` check to avoid losses on;
[Erc20Quest.sol#L81-L86](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81-L86)

## `initialize` functions can be frontrun

The `initialize` function that initializes important contract state can be called by anyone.  
See:
- [QuestFactory.sol#L37](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37)
- [RabbitHoleReceipt.sol#L43](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L43)
- [RabbitHoleTickets.sol#L32](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L32)

An unauthorized party can initialize the contract before the legitimate deployer, hoping that the victim continues to use the same contract.  
In the best case for the victim, they notice it and have to redeploy their contract costing gas.

For initializing proxy contracts, deploy contracts using a factory contract that immediately calls `initialize` after deployment or make sure to call it immediately after deployment and verify the transaction succeeded.


# QA

## Contract `Quest` should be marked as abstract

If you set `Quest` as abstract you could avoid create empty functions and just enforce the contracts using `Quest` to implement them;

```diff
diff --git a/contracts/Erc1155Quest.sol b/contracts/Erc1155Quest.sol
index 4811bc9..6ff8dd3 100644
--- a/contracts/Erc1155Quest.sol
+++ b/contracts/Erc1155Quest.sol
@@ -51,8 +51,7 @@ contract Erc1155Quest is Quest, ERC1155Holder {
 
     /// @dev Withdraws the remaining tokens from the contract. Only able to be called by owner
     /// @param to_ The address to send the remaining tokens to
-    function withdrawRemainingTokens(address to_) public override onlyOwner {
-        super.withdrawRemainingTokens(to_);
+    function withdrawRemainingTokens(address to_) public override onlyOwner onlyAdminWithdrawAfterEnd {
         IERC1155(rewardToken).safeTransferFrom(
             address(this),
             to_,
diff --git a/contracts/Erc20Quest.sol b/contracts/Erc20Quest.sol
index 868a790..4af9709 100644
--- a/contracts/Erc20Quest.sol
+++ b/contracts/Erc20Quest.sol
@@ -78,8 +78,7 @@ contract Erc20Quest is Quest {
     /// @notice Function that allows the owner to withdraw the remaining tokens in the contract
     /// @dev Every receipt minted should still be able to claim rewards (and cannot be withdrawn). This function can only be called after the quest end time
     /// @param to_ The address to send the remaining tokens to
-    function withdrawRemainingTokens(address to_) public override onlyOwner {
-        super.withdrawRemainingTokens(to_);
+    function withdrawRemainingTokens(address to_) public override onlyOwner onlyAdminWithdrawAfterEnd {
 
         uint unclaimedTokens = (receiptRedeemers() - redeemedTokens) * rewardAmountInWeiOrTokenId;
         uint256 nonClaimableTokens = IERC20(rewardToken).balanceOf(address(this)) - protocolFee() - unclaimedTokens;
diff --git a/contracts/Quest.sol b/contracts/Quest.sol
index 33163ee..2a14d8c 100644
--- a/contracts/Quest.sol
+++ b/contracts/Quest.sol
@@ -9,7 +9,7 @@ import {ECDSA} from '@openzeppelin/contracts/utils/cryptography/ECDSA.sol';
 /// @title Quest
 /// @author RabbitHole.gg
 /// @notice This contract is the base contract for all Quests. The Erc20Quest and Erc1155Quest contracts inherit from this contract.
-contract Quest is Ownable, IQuest {
+abstract contract Quest is Ownable, IQuest {
     RabbitHoleReceipt public immutable rabbitHoleReceiptContract;
     address public immutable rewardToken;
     uint256 public immutable endTime;
@@ -119,16 +119,12 @@ contract Quest is Ownable, IQuest {
 
     /// @notice Calculate the amount of rewards
     /// @dev This function must be implemented in the child contracts
-    function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {
-        revert MustImplementInChild();
-    }
+    function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256);
 
     /// @notice Transfer the rewards to the user
     /// @dev This function must be implemented in the child contracts
     /// @param amount_ The amount of rewards to transfer
-    function _transferRewards(uint256 amount_) internal virtual {
-        revert MustImplementInChild();
-    }
+    function _transferRewards(uint256 amount_) internal virtual;
 
     /// @notice Checks if a Receipt token id has been used to claim a reward
     /// @param tokenId_ The token id to check
@@ -147,5 +143,5 @@ contract Quest is Ownable, IQuest {
     }
 
     /// @notice Allows the owner of the Quest to withdraw any remaining rewards after the Quest has ended
-    function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
+    function withdrawRemainingTokens(address to_) public virtual;
```


## Bad naming of functions `onlyAdminWithdrawAfterEnd` is not doing a `onlyAdmin`
Beware of this kind of function in a future someone could interpretate that its doing a onlyAdmin check, but it only checks for `endTime` if this happen it could be a big issue;
```solidity
/// @notice Prevents reward withdrawal until the Quest has ended
modifier onlyAdminWithdrawAfterEnd() {
    if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
    _;
}
```

I recommend you to rename this function from `onlyAdminWithdrawAfterEnd` to `onlyAfterEnd`

## Its preffer to use `_disableInitializer` on the constructor
Currently on [QuestFactory.sol#L34-L35](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L34-L35) you are using `initializer` on the constructor
```solidity
    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() initializer {}
```

The recomendation is to use `_disableInitializers` like you do on [RabbitHoleTickets.sol#L27-L30](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L27-L30)


## Remove unused imports

`ECDSA` its not being use on [Quest.sol#L7](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L7)

## Use `100_00` instead of `10_000` to represent `bps`
If you are talking about percentages, and `10_000` is 100% i think its more clear to use `100_00`

## Avoid magic numbers
The number `10_000` repeats all over the code, instead the raw number use a constant;
`uint16 MAX_BPS = 100_00;`

## Be consisten with the imports, use named imports
Instead of `import "Contract.sol";` use `import {Contract} from "Contract.sol";`
- [QuestFactory.sol#L4](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L4)
- [QuestFactory.sol#L10](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L10)
- [QuestFactory.sol#L11](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L11)
- [RabbitHoleReceipt.sol#L4-L13](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L4-L13)
- [RabbitHoleTickets.sol#L4-L9](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L4-L9)
- [ReceiptRenderer.sol#L4-L5](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L4-L5)
- [TicketRenderer.sol#L4-L5](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L4-L5)

## Prefer `uint256` instead of `uint`
`uint` is an alias of `uint256`, but its better to always specify that you are using `uint256`

## Avoid params verbosity thought `struct` to pack data config
A common pattern to avoid variables repetition and stack too deep its to pack everything in a struct, in RabbitHole cases i think that packing all the quest variables into a struct its an excellent idea. 

```diff
diff --git a/contracts/Erc1155Quest.sol b/contracts/Erc1155Quest.sol
index 4811bc9..2f2d2cb 100644
--- a/contracts/Erc1155Quest.sol
+++ b/contracts/Erc1155Quest.sol
@@ -11,23 +11,8 @@ import {Quest} from './Quest.sol';
 contract Erc1155Quest is Quest, ERC1155Holder {
 
     constructor(
-        address rewardTokenAddress_,
-        uint256 endTime_,
-        uint256 startTime_,
-        uint256 totalParticipants_,
-        uint256 rewardAmountInWeiOrTokenId_,
-        string memory questId_,
-        address receiptContractAddress_
-    )
-    Quest(
-        rewardTokenAddress_,
-        endTime_,
-        startTime_,
-        totalParticipants_,
-        rewardAmountInWeiOrTokenId_,
-        questId_,
-        receiptContractAddress_
-    ){}
+        QuestConfig memory questConfig
+    ) Quest(questConfig){}
 
     /// @dev Checks the balance to ensure that it has enough for all of the participants. Only able to be called by owner
     function start() public override {
diff --git a/contracts/Erc20Quest.sol b/contracts/Erc20Quest.sol
index 868a790..fb7daf7 100644
--- a/contracts/Erc20Quest.sol
+++ b/contracts/Erc20Quest.sol
@@ -15,25 +15,11 @@ contract Erc20Quest is Quest {
     QuestFactory public immutable questFactoryContract;
 
     constructor(
-        address rewardTokenAddress_,
-        uint256 endTime_,
-        uint256 startTime_,
-        uint256 totalParticipants_,
-        uint256 rewardAmountInWeiOrTokenId_1;
        
-            rewardAmountInWeiOrTokenId_,
-            questId_,
-            receiptContractAddress_
-        )
+        Quest(questConfig)
     {
         questFee = questFee_;
         protocolFeeRecipient = protocolFeeRecipient_;
diff --git a/contracts/Quest.sol b/contracts/Quest.sol
index 33163ee..647bbbb 100644
--- a/contracts/Quest.sol
+++ b/contracts/Quest.sol
@@ -23,25 +23,28 @@ contract Quest is Ownable, IQuest {
 
     mapping(uint256 => bool) private claimedList;
 
+    struct QuestConfig {
+        address rewardTokenAddress;
+        uint256 endTime;
+        uint256 startTime;
+        uint256 totalParticipants;
+        uint256 rewardAmountInWeiOrTokenId;
+        string questId;
+        address receiptContractAddress;
+    }
     constructor(
-        address rewardTokenAddress_,
-        uint256 endTime_,
-        uint256 startTime_,
-        uint256 totalParticipants_,
-        uint256 rewardAmountInWeiOrTokenId_,
-        string memory questId_,
-        address receiptContractAddress_
+        QuestConfig memory questConfig
     ) {
-        if (endTime_ <= block.timestamp) revert EndTimeInPast();
-        if (startTime_ <= block.timestamp) revert StartTimeInPast();
-        if (endTime_ <= startTime_) revert EndTimeLessThanOrEqualToStartTime();
-        endTime = endTime_;
-        startTime = startTime_;
-        rewardToken = rewardTokenAddress_;
-        totalParticipants = totalParticipants_;
-        rewardAmountInWeiOrTokenId = rewardAmountInWeiOrTokenId_;
-        questId = questId_;
-        rabbitHoleReceiptContract = RabbitHoleReceipt(receiptContractAddress_);
+        if (questConfig.endTime <= block.timestamp) revert EndTimeInPast();
+        if (questConfig.startTime <= block.timestamp) revert StartTimeInPast();
+        if (questConfig.endTime <= questConfig.startTime) revert EndTimeLessThanOrEqualToStartTime();
+        endTime = questConfig.endTime;
+        startTime = questConfig.startTime;
+        rewardToken = questConfig.rewardTokenAddress;
+        totalParticipants = questConfig.totalParticipants;
+        rewardAmountInWeiOrTokenId = questConfig.rewardAmountInWeiOrTokenId;
+        questId = questConfig.questId;
+        rabbitHoleReceiptContract = RabbitHoleReceipt(questConfig.receiptContractAddress);
         redeemedTokens = 0;
     }
 
diff --git a/contracts/QuestFactory.sol b/contracts/QuestFactory.sol
index cfba7f5..8e2a03f 100644
--- a/contracts/QuestFactory.sol
+++ b/contracts/QuestFactory.sol
@@ -3,6 +3,7 @@ pragma solidity ^0.8.15;
 
 import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
 import {Erc20Quest} from './Erc20Quest.sol';
+import {Quest as QuestContract} from './Quest.sol';
 import {IQuestFactory} from './interfaces/IQuestFactory.sol';
 import {Erc1155Quest} from './Erc1155Quest.sol';
 import {RabbitHoleReceipt} from './RabbitHoleReceipt.sol';
@@ -69,17 +70,21 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
     ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
         if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();
 
+        QuestContract.QuestConfig memory questConfig = QuestContract.QuestConfig(
+            rewardTokenAddress_,
+            endTime_,
+            startTime_,
+            totalParticipants_,
+            rewardAmountOrTokenId_,
+            questId_,
+            address(rabbitholeReceiptContract)
+        );
+
         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
             if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
 
             Erc20Quest newQuest = new Erc20Quest(
-                rewardTokenAddress_,
-                endTime_,
-                startTime_,
-                totalParticipants_,
-                rewardAmountOrTokenId_,
-                questId_,
-                address(rabbitholeReceiptContract),
+                questConfig,
                 questFee,
                 protocolFeeRecipient
             );
@@ -87,13 +92,7 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
             emit QuestCreated(
                 msg.sender,
                 address(newQuest),
-                questId_,
-                contractType_,
-                rewardTokenAddress_,
-                endTime_,
-                startTime_,
-                totalParticipants_,
-                rewardAmountOrTokenId_
+                questConfig
             );
             quests[questId_].questAddress = address(newQuest);
             quests[questId_].totalParticipants = totalParticipants_;
@@ -106,25 +105,13 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
             if (msg.sender != owner()) revert OnlyOwnerCanCreate1155Quest();
 
             Erc1155Quest newQuest = new Erc1155Quest(
-                rewardTokenAddress_,
-                endTime_,
-                startTime_,
-                totalParticipants_,
-                rewardAmountOrTokenId_,
-                questId_,
-                address(rabbitholeReceiptContract)
+                questConfig
             );
 
             emit QuestCreated(
                 msg.sender,
                 address(newQuest),
-                questId_,
-                contractType_,
-                rewardTokenAddress_,
-                endTime_,
-                startTime_,
-                totalParticipants_,
-                rewardAmountOrTokenId_
+                questConfig
             );
             quests[questId_].questAddress = address(newQuest);
             quests[questId_].totalParticipants = totalParticipants_;
diff --git a/contracts/interfaces/IQuestFactory.sol b/contracts/interfaces/IQuestFactory.sol
index b93008b..0491446 100644
--- a/contracts/interfaces/IQuestFactory.sol
+++ b/contracts/interfaces/IQuestFactory.sol
@@ -1,5 +1,6 @@
 // SPDX-License-Identifier: UNLICENSED
 pragma solidity ^0.8.15;
+import {Quest} from "../Quest.sol";
 
 interface IQuestFactory {
     error QuestIdUsed();
@@ -13,7 +14,7 @@ interface IQuestFactory {
     error AddressZeroNotAllowed();
     error QuestFeeTooHigh();
 
-    event QuestCreated(address indexed creator, address indexed contractAddress, string indexed questId, string contractType, address rewardTokenAddress, uint256 endTime, uint256 startTime, uint256 totalParticipants, uint256 rewardAmountOrTokenId);
+    event QuestCreated(address indexed creator, address indexed contractAddress, Quest.QuestConfig questConfig);
     event ReceiptMinted(address indexed recipient, string indexed questId);
 
     function questInfo(string memory questId_) external view returns (address, uint, uint);
```

## Missing functions on interface `IQuest`

There are some missing functions on [`IQuest`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol) such as;
- `start()`
- `pause()`
- `unpause()`
- `getRewardAmount()`
- `withdrawRemainingTokens(address to_)`

## Missing event emission on critical functions
On [QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol), the critical functions should emit events notifing the updates;
`setClaimSignerAddress`, `setProtocolFeeRecipient`, `setRabbitHoleReceiptContract`, `setRewardAllowlistAddress`, `setQuestFee`

On [Erc20Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol): `withdrawFee`
On [Erc1155Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol): `withdrawRemainingTokens`
On [RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol): `setReceiptRenderer`, `setRoyaltyRecipient`, `setQuestFactory`
On [RabbitHoleTickets.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol): `setTicketRenderer`, `setRoyaltyRecipient`

