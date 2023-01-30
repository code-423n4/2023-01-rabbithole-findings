1.  Separating Emits for Quest Creation between ERC20 and ERC1155 Tokens

This was happen in fn createQuest() [L87-97](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L87-L97) and [L.118-128](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L118-L128).

Emitting multiple events from the same function can cause ambiguity and make it difficult to distinguish between the different events. It's recommended to have separate functions for emitting each type of event, so that they can be clearly distinguished. This makes the code more readable, maintainable, and easier to understand.

In order to make it different, you can create two separate events for each token type: one for ERC20 and one for ERC1155. This way, you can track the creation of each type of token in a distinct and specific manner. It can be implemented as follows:

Set it [from](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L16):
```
event ERC20QuestCreated(...);
event ERC1155QuestCreated(...);
```
and emitting : 
```
    emit ERC20QuestCreated(....);
    emit ERC1155QuestCreated(....);
```

2. Unused Code: AmountExceedsBalance() Function in Quest.sol

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L18

```
    error AmountExceedsBalance();
```

Handling error was set but never called at `Quest.sol`, `Erc20Quest.sol`,`Erc1155Quest.sol` on any other function. The "error AmountExceedsBalance()" should be placed in the function that handles the transaction, such as the `claim` function, to prevent the transfer of more tokens than the user's available balance. It can be placed before the transfer of tokens to ensure that the user's balance is sufficient before proceeding with the transfer.

3. Implementing QuestFeeMinimum to Prevent Setting Zero Quest Fee in QuestFactory Contract

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L183-L189

```
function setQuestFee(uint256 questFee_) public onlyOwner {
    if (questFee_ > 10_000) revert QuestFeeTooHigh();
    if (questFee_ < 1) revert QuestFeeTooLow();
    questFee = questFee_;
}

```

we've added a check to see if the quest fee is less than 1. If it is, we revert with an error message `QuestFeeTooLow`. This ensures that the `questFee` can never be set to zero or a negative value.

4. TYPO

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L176

`remave` into `remove`
