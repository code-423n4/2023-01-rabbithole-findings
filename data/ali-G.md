# 2023-01-rabbithole-Findings

# Gas

## [G-1] Using Private Rather Than Public for Constants

Saves gas in deployment due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table

Saves `11238 gas` on deployment

*There is 1 instance of this issue:*

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L17

Can be changed like this:

```solidity
    // @audit gas 
    // deployment  5268486 -> 5257248 = -11238 gas saved
    bytes32 private constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
```


## [G-2] Local Variable Assignment

Cache frequently used storage variables in memory/stack, converting multiple SLOAD into 1 SLOAD.

Saves `13398 - 11598 gas` on deployment

Saves `716 gas` on function createQuest call

*There are 12 instances of this issue:*

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L70
Can be changed like this:

```solidity
        Quest storage quest = quests[questId_];
        if (quest.questAddress != address(0)) revert QuestIdUsed();
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L98
Can be changed like this:

```solidity
quest.questAddress = address(newQuest);
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L99
Can be changed like this:

```solidity
quest.totalParticipants = totalParticipants_;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L129
Can be changed like this:

```solidity
quest.questAddress = address(newQuest);
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L130
Can be changed like this:

```solidity
quest.totalParticipants = totalParticipants_;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L201

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L202

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L203
```solidity
Deployment cost 
before: 5300118 
after: 5288520
```
Saves `11598 gas` on deployment

Can be changed like this:

```solidity
        // @audit gas Quest storage quest = quests[questId_];
        // deployment  5300118 -> 5288520 = -11598 gas saved
        Quest storage quest = quests[questId_];
        return (
            quest.questAddress,
            quest.totalParticipants,
            quest.numberMinted
        );
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L220
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L225
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L226
```solidity
Deployment cost 
before: 5251239
after: 5231184
```
Saves `20055 gas` on deployment
```solidity
Function calling cost
before: 288075
after: 287092
```
Saves `983 gas` on function call

Can be changed like this:
```solidity
        Quest storage quest = quests[questId_];
```
```solidity
        if (quest.numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
        if (quest.addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```
```solidity
        quest.addressMinted[msg.sender] = true;
        quest.numberMinted++;
```

## [G-3] `>=` is cheaper than `>`

Non-strict inequalities `>=` are cheaper than strict ones `>`. This is due to some supplementary checks (ISZERO, 3 gas)).

*There are 3 instances of this issue:*

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L187
```solidity
Deployment cost 
before: 5234829 
after: 5234589
```
Saves `240 gas` on deployment
```solidity
Function calling cost
before: 36037
after: 36034
```
Saves `3 gas` on function call


Can be changed like this:

```solidity
        if (questFee_ >= 10_001) revert QuestFeeTooHigh();
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L220
```solidity
Deployment cost 
before: 5234589 
after: 5231340
```
Saves `3249 gas` on deployment
```solidity
Function calling cost
before: 287092
after: 287016
```
Saves `76 gas` on function call


Can be changed like this:

```solidity
        if (quest.numberMinted >= quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L129
```solidity
Deployment cost 
before: 2761124 
after: 2760896
```
Saves `228 gas` on deployment

Can be changed like this:

```solidity
            if (tokenIdsForQuest[i] >= 1) {
```

## [G-4] Use Assembly to Write Storage Values

*There are 8 instances of this issue:*

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L160
```solidity
Deployment cost 
before: 5246034 
after: 5234829
```
Saves `11205 gas` on deployment
```solidity
Function calling cost
before: 36313
after: 36262
```
Saves `51 gas` on function call


Can be changed like this:

```solidity
        assembly{
            sstore(claimSignerAddress.slot, claimSignerAddress_)
        }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L167
```solidity
Deployment cost 
before: 5257248 
after: 5246034
```
Saves `11214 gas` on deployment
```solidity
Function calling cost
before: 36369
after: 36318
```
Saves `51 gas` on function call


Can be changed like this:

```solidity
        assembly{
            sstore(protocolFeeRecipient.slot, protocolFeeRecipient_)
        }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L72
```solidity
Deployment cost 
before: 2776002 
after: 2766963
```
Saves `9039 gas` on deployment

Can be changed like this:

```solidity
        assembly{
            sstore(royaltyRecipient.slot, royaltyRecipient_)
        }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L84
```solidity
Deployment cost 
before: 2766963 
after: 2761124
```
Saves `5839 gas` on deployment

Can be changed like this:

```solidity
        assembly{
            sstore(minterAddress.slot, minterAddress_)
        }
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90

Can be changed like this:

```solidity
        assembly{
            sstore(royaltyFee.slot, royaltyFee_)
        }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L61
```solidity
Deployment cost 
before: 2623629 
after: 2612418
```
Saves `11211 gas` on deployment

Can be changed like this:

```solidity
        assembly{
            sstore(royaltyRecipient.slot, royaltyRecipient_)
        }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L67

Can be changed like this:

```solidity
        assembly{
            sstore(royaltyFee.slot, royaltyFee_)
        }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L74
```solidity
Deployment cost 
before: 2612418 
after: 2604455
```
Saves `7963 gas` on deployment

Can be changed like this:

```solidity
        assembly{
            sstore(minterAddress.slot, minterAddress_)
        }
```

## [G-5] Remove Initializing Variables with Default Value

Explicitly initializing a variable with it's default value costs unnecessary gas.

*There are 7 instances of this issue:*

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L115

Can be changed like this:

```solidity
        uint foundTokens;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L126

Can be changed like this:

```solidity
        uint filterTokensIndexTracker;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117

Can be changed like this:

```solidity
        for (uint i; i < msgSenderBalance; i++)
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128

Can be changed like this:

```solidity
        for (uint i; i < msgSenderBalance; i++)
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L45

Can be changed like this:

```solidity
        redeemedTokens;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70

Can be changed like this:

```solidity
        for (uint i; i < tokenIds_.length; i++) {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104

Can be changed like this:

```solidity
        for (uint i; i < tokens.length; i++) {
```

## [G-6] Unchecked Loop Index Increase

*There are 4 instances of this issue:*

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117
```solidity
Deployment cost 
before: 2760896 
after: 2758760
```
Saves `2136 gas` on deployment

Can be changed like this:

```solidity
        for (uint i; i < msgSenderBalance;) { // @audit
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
            }
            unchecked {
                ++i;
            }
        }
```


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128
```solidity
Deployment cost 
before: 2758760 
after: 2756564
```
Saves `2196 gas` on deployment

Can be changed like this:

```solidity
        for (uint i; i < msgSenderBalance;) { // @audit
            if (tokenIdsForQuest[i] >= 1) {
                filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
                filterTokensIndexTracker++;
            }
            unchecked {
                ++i;
            }
        }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70

Can be changed like this:

```solidity
        for (uint i; i < tokenIds_.length;) {
            claimedList[tokenIds_[i]] = true;
            unchecked {
                ++i;
            }
        }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104

Can be changed like this:

```solidity
        for (uint i; i < tokens.length;) {
            if (!isClaimed(tokens[i])) {
                redeemableTokenCount++;
            }
            unchecked {
                ++i;
            }
        }
```

## [G-7] Use Prefix Increment instead of Postfix Increment

The difference between the prefix increment and postfix increment expression lies in the return value of the expression.

The prefix increment expression (++i) returns the updated value after it's incremented. The postfix increment expression (i++) returns the original value.

The prefix increment expression is cheaper in terms of gas.

Consider using the prefix increment expression whenever the return value is not needed.


*There are 3 instances of this issue:*

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L121
```solidity
Deployment cost 
before: 2756564 
after: 2756156
```
Saves `408 gas` on deployment

Can be changed like this:

```solidity
                ++foundTokens;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L131
```solidity
Deployment cost 
before: 2756156 
after: 2755736
```
Saves `420 gas` on deployment

Can be changed like this:

```solidity
                ++filterTokensIndexTracker;
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L106

Can be changed like this:

```solidity
                ++redeemableTokenCount;
```

## [G-8] Use Assembly for Math

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

*There is 1 instance of this issue:*

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L113
```solidity
Deployment cost 
before: 2604455 
after: 2602931
```
Saves `1524 gas` on deployment

Can be changed like this:

```solidity
        assembly {
            let c := mul(salePrice_, royaltyFee.slot)

            if lt(c, salePrice_) {
                mstore(0x00, "overflow")
                revert(0x00, 0x20)
            }

            royaltyPayment := div(c, 10000)

            if gt(royaltyPayment, c) {
                mstore(0x00, "underflow")
                revert(0x00, 0x20)
            }
        }
```

## [G-9] Cache Array Length Outside of Loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

*There is 1 instance of this issue:*

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70

```solidity
Deployment cost 
before: 5231184 
after: 5230968
```
Saves `216 gas` on deployment
```solidity
Function calling cost
before: 1500237
after: 1499669
```
Saves `568 gas` on function call

Can be changed like this:

```solidity
        uint256 len = tokenIds_.length;
        for (uint i; i < len;) {
            claimedList[tokenIds_[i]] = true;
            unchecked {
                ++i;
            }
        }
```

## [G-10] X += Y Costs More Gas Than X = X + Y For State Variables

*There is 1 instance of this issue:*

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115

Can be changed like this:

```solidity
        redeemedTokens = redeemedTokens + redeemableTokenCount;
```

## TOTAL GAS SAVED

These changes saves more than ~122k gas in total.