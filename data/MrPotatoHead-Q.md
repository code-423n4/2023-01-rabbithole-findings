1. Incorrect variable name: rewardAmountInWeiOrTokenId 

[https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L46](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L46)

The variable rewardAmountInWeiOrTokenId has to be a monetary value for the accounting to work further on.
I'm assuming that reward amount in wei is of a far greater value than a tokenId and if that's true then the starting function, later on, will start even when the balance on the contract is less than the amount needed for the awards.

Recommendation:

Change the variable name to rewardAmountInWei. As to not confuse the person calling the Quest

2. No checks on the tokenId.length

[https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L69](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L46)

The for loop inside the function loops the length of the tokenIds, which is input by the admin, however, there needs to be a check that the tokenId.length doesn't exceed the gas limit

3. No checks for minting Id’s

[https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTicket.sol#L92](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L46)

No checks the input Ids are accurate meaning the minter can mint tokens for incorrect ids_

4. Some contracts use uint and some use uint256:

Look in RabitHoleReceipt contract.