1) Use calldata instead of memory for function parameters( QuestFactory.sol - 199, 193; Quest.sol - Line 69-73);
IQuestFactory.sol use calldata with questInfo() params RabbitHoleTickets.sol mint finction use calldata


2) Pack struct(QuestFactory.sol)
address = 20 bytes; mapping - ?; uint = 32 bytes; put variables in struct in such order: uint; uint; address; mapping;

3) make questFee immutable (QuestFactory.sol - line 31)

4) make functions internal  (QuestFactory.sol - setQuestFee_ can be internal; make recoverSigner internal) (Erc20Quest.sol - protocolFee internal;  receiptRedeemers internal; maxProtocolReward internal ;) (Quest.sol isClaimed internal) 

5) use ++quests[questId_].numberMinted; instead of quests[questId_].numberMinted++; (QuestFactory.sol - line 226)

6) Initialize questId as bytes

7) Pack booleans

8) IQuest.sol use indexed in events

9) RabbitHoleTickets.sol line 122 exchange like that: super.supportsInterface(interfaceId_); || type(IERC2981Upgradeable).interfaceId

10) add unchecked in protocolFee, calculateRewards Erc20Quest.sol;

Quest.sol - add unchecked (line 106) & unchecked for ++i in lines 104-108
add unchecked for ++i in lines 70-73

add unchecked line 132, 226 and put "++" in front of variable - QuestFactory.sol

RabbitHoleReceipt.sol - unchecked for ++i  and unchecked{++_variableName} line 117-123 and the same on lines 128-133




