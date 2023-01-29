## Quest.sol

Link: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L75-L79
The naming of the modifier can lead to confusion, since it is has nothing to do with the admin, anyone can call a function with this modifier, even if it does not lead to bigger vulnerabilities.

Link: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L7
Unused import, could be removed

Link: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L114-L115
I would suggest updating the state variable redeemedTokens before transferring them.

## QuestFactory.sol

Link: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L29
I suggest to declare it as an address and cast it when calling the mint() function because the casting to address is done a lot of times. I think this will reduce a little bit of gas.

Link: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L44
I wouldn't create an internal function with just two lines just to be called once in the initialize. I suggest to remove the function and just grant the roles in the constructor.

Link: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L68
I suggest to use a uint instead of a string to identify the quests, something similar to the Counter in the ERC721 from OpenZeppelin.

Link: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L67
I suggest to introduce the contract type as boolean instead of a string. This way the if statement would be easier. Also, the last revert (revert QuestTypeInvalid();) would not be necessary. The event QuestCreated could be emitted outside the if statement. All this would lead to shorter, more readable and cheaper code. 

Link: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L139-L148
I suggest to use the functions the library of AccessControl provides directly, instead of creating a function for calling this functions. It is probably cheaper and innecesary code can be removed, making a more readable contract. 

Link: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L227-L228
I suggest to emit the event after the mint function is called.

## Every contract

This is something that happens in every contract:
From the article Security Pitfalls And Best practices 101 from Secureum: Unlocked pragma: Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the pragma (for e.g. by not using ^ in pragma solidity 0.5.10) ensures that contracts do not accidentally get deployed using an older compiler version with unfixed bugs. 




