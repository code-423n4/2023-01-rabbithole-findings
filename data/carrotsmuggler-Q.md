# 1. ERC1155Quest.sol: withdrawRemainingTokens() does not leave tokens for users who completed the challenge but hasn't claimed their reward yet.
ERC20Quest.sol implements this function in a way where users can still claim their rewards after the owner clears out the quest contract. Suggest implementing the same logic for ERC1155Quest as well.

# 2. ERC1155Quest.sol has no recovery/destination address option
ERC721 and ERC1155 transfer contracts generally have a recovery address option allowing the msg.sender to send the ERC721/1155 token to a different address. This is recommended since not all smart contract wallets have the required callback functions implemented, and thus might lead to stuck tokens/unexpected reverts. Advised to add recovery address option, or take a destination address as an input.

# 3. QuestFactory.sol: mintReceipt() has no recovery/destination address option
ERC721 and ERC1155 transfer contracts generally have a recovery address option allowing the msg.sender to send the ERC721/1155 token to a different address. This is recommended since not all smart contract wallets have the required callback functions implemented, and thus might lead to stuck tokens/unexpected reverts. Advised to add recovery address option, or take a destination address as an input.

# 4. Quest.sol: start() should only be callable once
Currently, `start()` function can be called multiple times, and unpauses the contract as well. General practice is to have a `start()` function which is callable once, and use `pause/unpause` to control the contract instead of the start function.

# 5. Quest.sol: onlyAdminWithdrawAfterEnd() modifier does not check for admin
The modifier name says onlyAdmin, but the modifier itself doesn't check for admin. Consider renaming the modifier.

# 6. Quest.sol: Long list of minted receipts can cause out of gas error.
Consider limiting maximum number of claims in a single transaction.

# 7. QuestFactory.sol: Missing address validations when creating ERC1155Quests
ERC1155Quest creator does not validate the passed ERC1155 address. It is easy to validate ERC1155 address since they support ERC165 so a simple `supportInterface` call is advised.

# 8. QuestFactory.sol: totalparticipants_ should be required to be more than 0

# 9. Missing events
Every change of storage state in a contract should be accompanied by an event. This is missing in a few places as listed below
- Quest.sol
    - start()
    -pause()
    -unpause()

