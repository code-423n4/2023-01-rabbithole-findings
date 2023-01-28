## Gas Optimisation

1. Usage of eip 1167 and clone factory to drastically reduce gas fees for the users when createQuest is called.
You can use Openzeppelin's Clones.sol to do this.

2. Instead of using the function - grantDefaultAdminAndCreateQuestRole. Include the below lines in the initialize function to save gas
```
_grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
_grantRole(CREATE_QUEST_ROLE, msg.sender);
```

3. no need to initialize redeemedTokens = 0 in Quest.sol as it is already 0

4. In Quest.sol, Line 136, instead of return claimedList[tokenId_] == true, just use return claimedList[tokenId_]; 
the same in QuestFactory.sol as well - Line 221, Use quests[questId_].addressMinted[msg.sender]

5. In Quest.sol, Line 115 - instead of redeemedTokens += redeemableTokenCount, use redeemedTokens = redeemedTokens + redeemableTokenCount; as it saves some gas