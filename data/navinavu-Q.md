## QA - Use full type name (ex: uint256 instead of uint)
Use full types names for readability and consistency across the codebase.
`uint` is the alias for `uint256`. But in some scenarios, specifying the short name would lead to issues.
Example while computing function signatures:
`bytes4(keccak256('f(uint)'))` would return a different hash than `bytes4(keccak256('f(uint256)'))`.

### Context
Global

---

## QA - Revert CustomError does not match real revert reason in a specific case
If the `questId_` key does not exist inside the state quests mapping, the first assertion will revert with a `OverMaxAllowedToMint()` error,
which may lead to confusion and should instead revert with a meaningful Custom Error if the key is unknown.
For example: `UnknownQuestID(uint256 questID)`.

### Context
- QuestFactory.sol : minReceipt() : [l:219](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219)

---

## QA - Unnecessary use of `onlyOwner` modifier on overriding function
The `onlyOwner` modifier is not necessary as the call to `super.withdrawRemainingTokens(to_)` will import the `onlyOwner`
modifier attached to the parent's virtual function.

### Context
- ERC20Quest.sol : withdrawRemainingTokens() : [l:81](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81)
- ERC1155Quest.sol : withdrawRemainingTokens() : [l:54](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54)
