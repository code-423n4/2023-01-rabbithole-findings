# GAS


## Use a defined length for `questId` stick to `bytesN`, if you havent decided use `bytes32` 

>
> From Solidity docs:
> As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

## Use `immutable` on `questId`
Ok, in the current context is impossible, but if you follow the previous recommendation (use `bytesN` instead of `string memory`) you could set `questId` on [Quest.sol#L21](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L21) as `immutable`

## Duplicate functions
Since state variable `rewardToken` is public there is no need for the function `getRewardToken`. Same goes for state variable `rewardAmountInWeiOrTokenId`, its public, there is no need for the function `getRewardAmount`.

```diff
diff --git a/contracts/Quest.sol b/contracts/Quest.sol
index 33163ee..caa72ae 100644
--- a/contracts/Quest.sol
+++ b/contracts/Quest.sol
@@ -136,16 +136,6 @@ contract Quest is Ownable, IQuest {
         return claimedList[tokenId_] == true;
     }
 
-    /// @dev Returns the reward amount
-    function getRewardAmount() public view returns (uint256) {
-        return rewardAmountInWeiOrTokenId;
-    }
-
-    /// @dev Returns the reward token address
-    function getRewardToken() public view returns (address) {
-        return rewardToken;
-    }
-
     /// @notice Allows the owner of the Quest to withdraw any remaining rewards after the Quest has ended
     function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
 }
```

## Unnecesary check on `isClaimed`
There is no need to check `claimedList[tokenId_] == true`, `claimedList[tokenId_]` is a bool;
```diff
diff --git a/contracts/Quest.sol b/contracts/Quest.sol
index 33163ee..81a21c9 100644
--- a/contracts/Quest.sol
+++ b/contracts/Quest.sol
@@ -133,7 +133,7 @@ contract Quest is Ownable, IQuest {
     /// @notice Checks if a Receipt token id has been used to claim a reward
     /// @param tokenId_ The token id to check
     function isClaimed(uint256 tokenId_) public view returns (bool) {
-        return claimedList[tokenId_] == true;
+        return claimedList[tokenId_];
     }
 
     /// @dev Returns the reward amount
```

## Pass the owner on `createQuest` instead of `transferOwnership`
You could save gas avoiding unnesesary calls, instead of `transferOwnership` on [QuestFactory.sol#L100](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L100) just use `_transferOwnership(owner_)` on and pass the owner on the construction.

To achive hthis without a ` try removing local variables` error you should create a `struct` to pack data config, please review my Q/A report.

```diff
diff --git a/contracts/Quest.sol b/contracts/Quest.sol
index 33163ee..e4260c6 100644
--- a/contracts/Quest.sol
+++ b/contracts/Quest.sol
@@ -30,7 +30,8 @@ contract Quest is Ownable, IQuest {
         uint256 totalParticipants_,
         uint256 rewardAmountInWeiOrTokenId_,
         string memory questId_,
-        address receiptContractAddress_
+        address receiptContractAddress_,
+        address owner_
     ) {
         if (endTime_ <= block.timestamp) revert EndTimeInPast();
         if (startTime_ <= block.timestamp) revert StartTimeInPast();
@@ -43,6 +44,7 @@ contract Quest is Ownable, IQuest {
         questId = questId_;
         rabbitHoleReceiptContract = RabbitHoleReceipt(receiptContractAddress_);
         redeemedTokens = 0;
+        _transferOwnership(owner_);
     }
 
     /// @notice Starts the Quest
```


## Avoid setting default variables
Remove [Quest.sol#L45](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L45), the default value for `redeemedTokens` is 0

## Remove unused variable `timestampForTokenId`
It seems that there isnt any contract using the variable on `timestampForTokenId`:
- [RabbitHoleReceipt.sol#L34](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L34)
- [RabbitHoleReceipt.sol#L102](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L102)

If the purpose of the variable is to provide the minted creation time for frontend display, it is recommended to utilize an off-chain method such as the graph or a custom indexer. This will reduce the need for storing unnecessary data on-chain and improve contract performance.

## Use `uint16` to represent `royaltyFee` instead of `uint256`
`royaltyFee` shouldnt be greater than `100_00`, this mean that you could use a lower interger to represent it, and take advantage off the gas savings compacting storage layout.

## Use `uint40` to represent timstamp for variables like `startTime` and `endTime` instead of `uint256`
You could safely represent time in a `uint40` instead of using `uint256` and take advantage off the gas savings compacting storage layout.

## Avoid comparission to false use `!val` isntead of `val == false`
```diff
diff --git a/contracts/QuestFactory.sol b/contracts/QuestFactory.sol
index cfba7f5..1e22125 100644
--- a/contracts/QuestFactory.sol
+++ b/contracts/QuestFactory.sol
@@ -70,7 +70,7 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
         if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();
 
         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
-            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
+            if (!rewardAllowlist[rewardTokenAddress_]) revert RewardNotAllowed();
```
