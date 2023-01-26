In the Erc20Quest.sol, the comment of the start function says: [...]  It also requires that the start time has passed; - but we don't have a variable to track the start time, so the owner can start the quest whenever he/she wants.

====

The getNumberMinted function from the QuestFactory.sol can be deleted. The dev can use the questInfo function to return the same value: quests[questId_].numberMinted

(, , uint256 a) = questInfo(index);
