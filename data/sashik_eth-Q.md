## L1 - Owner could withdraw all unclaimed tokens while some still should be claimable 

[```withdrawRemainingTokens()```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54)  function in the ```Erc1155Quest``` contract allows the owner to withdraw all remaining tokens, including unclaimed ones that may still be claimable in the future. This could result in the accidental withdrawal of tokens that are meant to remain on the contract balance until claimed by users.

```solidity
File: Erc1155Quest.sol
52:     /// @dev Withdraws the remaining tokens from the contract. Only able to be called by owner
53:     /// @param to_ The address to send the remaining tokens to
54:     function withdrawRemainingTokens(address to_) public override onlyOwner { 
55:         super.withdrawRemainingTokens(to_);
56:         IERC1155(rewardToken).safeTransferFrom(
57:             address(this),
58:             to_,
59:             rewardAmountInWeiOrTokenId,
60:             IERC1155(rewardToken).balanceOf(address(this), rewardAmountInWeiOrTokenId),
61:             '0x00'
62:         );
63:     }

```

*Recommended Mitigation Steps*

Consider adding tracking flow in ```Erc1155Quest``` contract ```withdrawRemainingTokens``` function similar to [```Erc20Quest```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81) withdrawing function:
```solidity
    function withdrawRemainingTokens(address to_) public override onlyOwner { 
        super.withdrawRemainingTokens(to_);
        uint256 remainingTokens = totalParticipants - questFactoryContract.getNumberMinted(questId);
        IERC1155(rewardToken).safeTransferFrom(
            address(this),
            to_,
            rewardAmountInWeiOrTokenId,
            remainingTokens,
            '0x00'
        );
    }
```

## L2 - ```mintReceipt()```  function lacks a check to verify if the quest has already ended

[```mintReceipt()```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219) function missing check for ended quest. This could result in a scenario where a receipt is minted after the quest has ended and the remaining tokens and fee have been withdrawn from the quest contract. This would result in a situation where there are more claimable receipts than there are tokens available in the quest balance to redeem them.

```solidity
File: QuestFactory.sol
215:     /// @dev mint a RabbitHole Receipt. Note: this contract must be set as Minter on the receipt contract
216:     /// @param questId_ The id of the quest
217:     /// @param hash_ The hash of the message
218:     /// @param signature_ The signature of the hash
219:     function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public { 
220:         if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint(); 
221:         if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted(); 
222:         if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
223:         if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned(); 
224: 
225:         quests[questId_].addressMinted[msg.sender] = true;
226:         quests[questId_].numberMinted++;
227:         emit ReceiptMinted(msg.sender, questId_);
228:         rabbitholeReceiptContract.mint(msg.sender, questId_);
229:     }

```

*Recommended Mitigation Steps*

Consider adding check for quest not ended in ```mintReceipt()``` function: 
```solidity
    function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public { 
        if (IQuest(quests[questId_].address).ended < block.timestamp) revert QuestEnded(); 
    	...
    }
```

## N01 - No need to compare boolean variable with ```true``` value

```solidity
File: QuestFactory.sol
221:         if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted(); 

File: Quest.sol
136:         return claimedList[tokenId_] == true; 
```

## N02 - Breaking CEI pattern
[```claim()```](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96) function in ```Quest``` contract breaks "Checks Effects Interactions" pattern, function calls to external contracts on line 114 (in child implementations) and change contract state after this call:

```solidity
File: Quest.sol
094:     /// @notice Allows user to claim the rewards entitled to them
095:     /// @dev User can claim based on the (unclaimed) number of tokens they own of the Quest
096:     function claim() public virtual onlyQuestActive {
097:         if (isPaused) revert QuestPaused();
098: 
099:         uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);
100: 
101:         if (tokens.length == 0) revert NoTokensToClaim();
102: 
103:         uint256 redeemableTokenCount = 0;
104:         for (uint i = 0; i < tokens.length; i++) {
105:             if (!isClaimed(tokens[i])) {
106:                 redeemableTokenCount++;
107:             }
108:         }
109: 
110:         if (redeemableTokenCount == 0) revert AlreadyClaimed();
111: 
112:         uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);
113:         _setClaimed(tokens);
114:         _transferRewards(totalRedeemableRewards);
115:         redeemedTokens += redeemableTokenCount; 
116: 
117:         emit Claimed(msg.sender, totalRedeemableRewards);
118:     }

```

It does not lead to vulnerabilities now, but could lead in future, better to prevent it now by updating state before external interactions:
```solidity
112:         uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);
113:         _setClaimed(tokens);
114:         redeemedTokens += redeemableTokenCount; 
115:         _transferRewards(totalRedeemableRewards);
```

## N03 - Open TODO 

```solidity
File: IQuest.sol
4: // TODO clean this whole thing up 
```