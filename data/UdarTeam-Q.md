# [Q-01] User can DoS himself

## Description
If a single address has certain amount of ```RabbitHoleReceipt``` tokens (receipts) - according to tests ~1050, when he tries to call ```claim``` function from ```Quest.sol``` it will always revert with 'Transaction ran out of gas' error. The reason for the error is that ```claim``` function loops over user's all tokens.

```solidity
uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);
...
for (uint i = 0; i < tokens.length; i++) {
  if (!isClaimed(tokens[i])) {
    redeemableTokenCount++;
  }
}
```

## PoC
Add a new test case in ```test/Erc20Quest.spec.ts``` file.

```
it('user can DoS himself (claim function)', async () => {
    const signers = await ethers.getSigners();
    const seller = signers[9]
    const buyer = signers[10];

    await deployedQuestContract.start()
    await ethers.provider.send('evm_increaseTime', [86400])

    // Ignore below function. It represents 1050 tokens from different users.
    for (let i = 0; i < 1050; ++i) {
      await deployedRabbitholeReceiptContract.connect(seller).mint(seller.address, questId)
    }

    const totalTokens = await deployedRabbitholeReceiptContract.getOwnedTokenIdsOfQuest(questId, seller.address);

    // Assume buyer gets 1050 tokens from different users, not one
    for (let i = 0; i < totalTokens.length; ++i) {
      await deployedRabbitholeReceiptContract.connect(seller).transferFrom(seller.address, buyer.address, totalTokens[i]);
    }

    // Now buyer has 1050 tokens
    // He calls claim to get the rewards, but function reverts with `Transaction ran out of gas` error everytime
    await deployedQuestContract.connect(buyer).claim()
})
```

## Mitigation Steps
The user can always generate a new address and transfer some of the funds there. But if he has many tokens (> 10000), he should manage dozen accounts which can be become difficult and time consuming.
Better approach is to add maximum token claim amount as function parameter or to check ```tokens``` array length to not exceed certain amount.

# [Q-02] Explicitly define uint size

## Description
Define uint sizes explicitly for better code readability and consistency.

## Code location
- Erc20Quest.sol: [L84](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L84)
- Quest.sol: [L70](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70), [L99](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L99), [L104](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104)
- QuestFactory.sol: [L22-L23](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L22-L23), [L31-L32](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L31-L32), [L193](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L193), [L199](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L199)
- RabbitHoleReceipt.sol: [L30](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L30), [L33-L34](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L33-L34), [L47](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L47), [L100](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L100), [L112](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L112), [L113-L115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L113-L115), [L117-L118](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117-L118), [L125-L126](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L125-L126), [L128](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c21658332bd652d8d/contracts/RabbitHoleReceipt.sol#L128), [L159](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L159), [L165](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L165), [L169](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L169)
- RabbitHoleTickets.sol: [L24](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L24), [L36](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L36), [L102](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L102)
- ReceiptRenderer.sol: [L22](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L22), [L24](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L24), [L26](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L26), [L41](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L41), [L43](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L43), [L45](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L45), [L100](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L100)
- TicketRenderer.sol: [L17](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L17), [L36](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L36)
- IQuestFactory.sol: [L19](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L19)

# [Q-03] Typo in comment

## Code location
[L176](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L176)