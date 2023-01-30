## `onlyOwner` modifier called two times for the same function `withdrawRemainingTokens`

In contract `ERC20Quest`, for function [Erc20Quest.sol#L81](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81) there is `onlyOwner` modifier, which also exists in the base contract [Quest.sol#L150](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150), which is called with `super.withdrawRemainingTokens(to_);` in line [Erc20Quest.sol#L83](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L83). 

The same situation is also in contract `ERC1155Quest` [Erc1155Quest.sol#L54](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54) where also the super or base contract is called [Erc1155Quest.sol#L55](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L55).

I suggest removing the `onlyOwner` modifier from [Erc20Quest.sol#L81](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81)  and also from [Erc1155Quest.sol#L54](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54).


##  Optimize ` + 1 > ` with ` >= `

At line [QuestFactory.sol#L220](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L220) there is if check `if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();` which could be optimize with `(quests[questId_].numberMinted >= quests[questId_].totalParticipants) revert OverMaxAllowedToMint(); `. In that way, it will be one operation (add operation) less regarding the original code.



##  Not needed to set `isPaused = false;` because `isPaused` is by default false

At line [Quest.sol#L51](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L51) it is not needed `isPaused = false;` because by default bool storage variables are set to `false`, and on the other side, it is not possible to change that variable to `true` before the quest starts. So, I suggest removing that line.