## Use bitmaps to save gas

During calls to `claim`, the `claimedList` mapping is updated, setting the bool flag to true, but because the way the EVM works it allocates an entire storage slot (256 bits) every time a new flag is set. 

```javascript
function _setClaimed(uint256[] memory tokenIds_) private {
    for (uint i = 0; i < tokenIds_.length; i++) {
        claimedList[tokenIds_[i]] = true;
    }
}
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L69-L73

`SSTORE` opcode cost up to 20000 gas for uninitialized slots like these. Consider using bitmaps instead, this enables you to convert the `mapping(uint256 => bool)` to a `mapping(uint256 => uint256)` and pay the cost of slot initialization just once every 256 `tokenIds` added, so the high gas costs are amortized over many calls. Each group of 256 sequential `tokenIds` values (0-255, 256-511, ...) are stored together in a single uint256, where each bit represents a bool.

The full extent of the gas reduction from these changes cannot be seen using the tests present in the repo, as during the tests `claim` are called at most a few times, so the effect of amortization cannot be seen, underestimating the effect of these changes. Extra tests were added to enable benchmarking more realistic conditions (See modifications section below), where `claim` is called 256 times in sequence, simulating many user interacting with the contract during normal operating conditions. Using those benchmarks, we obtain the following results.

| Method  | avg. gas (before modification) | avg. gas (after modification) | gas diff                   |
|---------|--------------------------------|-------------------------------|----------------------------|
| claim() | 7488051                        | 1940555                       | -5547496 (74.1% reduction) |

*Values obtained by running `npx hardhat test --grep benchmark` on the modified Erc20Quest.spec.ts containing the git patch below.
```diff
diff --git a/Erc20Quest.spec.ts.orig b/Erc20Quest.spec.ts
index d0626ee..e639f3d 100644
--- a/Erc20Quest.spec.ts.orig
+++ b/Erc20Quest.spec.ts
@@ -302,6 +302,19 @@ describe('Erc20Quest', async () => {
       await expect(deployedQuestContract.claim()).to.be.revertedWithCustomError(questContract, 'AlreadyClaimed')
       await ethers.provider.send('evm_increaseTime', [-86400])
     })
+
+    it('benchmark multiple claims', async () => {
+      for (let i = 0; i < 256; i++) {
+        await deployedRabbitholeReceiptContract.mint(owner.address, questId)
+      }
+      await deployedQuestContract.start()
+
+      await ethers.provider.send('evm_increaseTime', [86400])
+
+      await deployedQuestContract.claim()
+
+      await ethers.provider.send('evm_increaseTime', [-86400])
+    })
   })

   describe('withdrawRemainingTokens()', async () => {
```

### Modifications
Consider using [Openzepelling's Bitmap](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/structs/BitMaps.sol) as shown below.

```diff
diff --git a/Quest.sol.orig b/Quest.sol
index 33163ee..20a72cd 100644
--- a/Quest.sol.orig
+++ b/Quest.sol
@@ -5,11 +5,13 @@ import {Ownable} from '@openzeppelin/contracts/access/Ownable.sol';
 import {IQuest} from './interfaces/IQuest.sol';
 import {RabbitHoleReceipt} from './RabbitHoleReceipt.sol';
 import {ECDSA} from '@openzeppelin/contracts/utils/cryptography/ECDSA.sol';
+import "@openzeppelin/contracts/utils/structs/BitMaps.sol";

 /// @title Quest
 /// @author RabbitHole.gg
 /// @notice This contract is the base contract for all Quests. The Erc20Quest and Erc1155Quest contracts inherit from this contract.
 contract Quest is Ownable, IQuest {
+    using BitMaps for BitMaps.BitMap;
     RabbitHoleReceipt public immutable rabbitHoleReceiptContract;
     address public immutable rewardToken;
     uint256 public immutable endTime;
@@ -21,7 +23,7 @@ contract Quest is Ownable, IQuest {
     string public questId;
     uint256 public redeemedTokens;

-    mapping(uint256 => bool) private claimedList;
+    BitMaps.BitMap private claimedList;

     constructor(
         address rewardTokenAddress_,
@@ -68,7 +70,7 @@ contract Quest is Ownable, IQuest {
     /// @param tokenIds_ The token ids to mark as claimed
     function _setClaimed(uint256[] memory tokenIds_) private {
         for (uint i = 0; i < tokenIds_.length; i++) {
-            claimedList[tokenIds_[i]] = true;
+            claimedList.set(tokenIds_[i]);
         }
     }

@@ -133,7 +135,7 @@ contract Quest is Ownable, IQuest {
     /// @notice Checks if a Receipt token id has been used to claim a reward
     /// @param tokenId_ The token id to check
     function isClaimed(uint256 tokenId_) public view returns (bool) {
-        return claimedList[tokenId_] == true;
+        return claimedList.get(tokenId_) == true;
     }

     /// @dev Returns the reward amount
```

