## [L-01] Receipt SVG not rendering correctly

`toString()` isn't called on `tokenId_` uint variable when rendering SVG for receipt.

```solidity
            '<text x="70%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #',
            tokenId_,
            '</text>',
```

Resulting base64 encoded image will not render as it will contain invalid characters.

Code reference:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L108-L110

## [L-02] Users can still mint receipts when quest is paused

Pausing functionality is unclear. After pausing a quest an owner has two options:
- Unpause and let users claim their rewards. In this case it's unclear why it was paused in the first place.
- Keep it paused until expiry. However, that doesn't prevent users from minting receipts. The owner won't be able to withdraw tokens allocated for rewards as it excludes the amount reserved for minted receipts:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L84-L85

## [L-03] DoS if address owns too many receipts

With time it is viable for users to acquire thousands and tens of thousands of receipts. This may happen as a result of buying receipts for example, which was highlighted as a valid use-case. Moreover, receipts aren't burned when they are used for claiming a reward.

Calculations in `getOwnedTokenIdsOfQuest` require looping over all of user's tokens. This may lead to denial of service as EVM isn't suitable for big loops.

```solidity
        uint msgSenderBalance = balanceOf(claimingAddress_);
        uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
        uint foundTokens = 0;

        for (uint i = 0; i < msgSenderBalance; i++) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
            }
        }
```

Recommendation: consider using ERC1155 for receipts which will allow tracking user receipts for each quest separately.

## [L-04] Changing `rabbitholeReceiptContract` in QuestFactory will break currently running quests

`rabbitHoleReceiptContract` must be the same in QuestFactory and Quest contracts for quests to function correctly. If there is a mismatch, then factory will mint receipts on a contract different from the one quests check when rewards are claimed.

Minting:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L228

Checking:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L99

QuestFactory has a function that allows changing the receipt contract. If that happens while quests are running, these quests will break.

Recommendation: don't allow changing receipt contract when there are active quests or document this scenario.

## [N-01] Make Quest contract abstract

Quest is a base class and should be made abstract as there is no intention to deploy it. This will also tidy up the code as empty function definitions can be changed to abstract:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L120-L131

## [N-02] Remove "== false" and "== true"

Comparing to true or false is redundant and can be removed:
```diff
+++ b/contracts/Quest.sol
@@ -133,7 +133,7 @@ contract Quest is Ownable, IQuest {
     /// @notice Checks if a Receipt token id has been used to claim a reward
     /// @param tokenId_ The token id to check
     function isClaimed(uint256 tokenId_) public view returns (bool) {
-        return claimedList[tokenId_] == true;
+        return claimedList[tokenId_];
     }
```

```diff
+++ b/contracts/QuestFactory.sol
@@ -70,7 +70,7 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
         if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();
 
         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
-            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
+            if (rewardAllowlist[rewardTokenAddress_]) revert RewardNotAllowed();
 
             Erc20Quest newQuest = new Erc20Quest(
                 rewardTokenAddress_,
@@ -218,7 +218,7 @@ contract QuestFactory is Initializable, OwnableUpgradeable, AccessControlUpgrade
     /// @param signature_ The signature of the hash
     function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
         if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
-        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
+        if (quests[questId_].addressMinted[msg.sender]) revert AddressAlreadyMinted();
         if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
         if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();
```

## [N-03] isPaused check can be moved to the modifier

The following check can be moved into `onlyQuestActive` or a modifier of its own. This will make the code cleaner.

```solidity
    function claim() public virtual onlyQuestActive {
        if (isPaused) revert QuestPaused();
```

Code reference:

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L97

## [N-04] Index correct event properties

To make it easier for external apps to integrate with the contracts, make sure events have indexed properties.

- Account can be indexed:
```solidity
event Claimed(address account, uint256 amount);
```
- contractType or startTime can be indexed instead of contractAddress
```solidity
event QuestCreated(address indexed creator, address indexed contractAddress, string indexed questId, string contractType, address rewardTokenAddress, uint256 endTime, uint256 startTime, uint256 totalParticipants, uint256 rewardAmountOrTokenId);
```