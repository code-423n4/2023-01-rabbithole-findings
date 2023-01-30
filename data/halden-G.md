# [01-GAS] Don not compare boolean expressions to boolean literals
It is more cheaper to not compare boolean expressions to boolean literals

QuestFactory:
73: `rewardAllowlist[rewardTokenAddress_] == false`

# [02-GAS] internal functions only called once can be inlined to save gas
# [03-GAS] Setting the constructor to payable
# [04-GAS] Using fixed bytes is cheaper than using string
For large name is more cheap to use fixed bytes length instead of string.

PoC
input: Very large string wich has more than 32 bytes and is the name of your quest

```
contract Test {

    mapping(string => uint256) test1;
    mapping(bytes32 => uint256) test2;

    function addInTest1(string calldata name) public {
        test1[name] = 1;
    }
    function addInTest2(string calldata name) public {
        test2[keccak256(abi.encode(name))] = 1;
    }
}
```
addInTest1: 52411 gas
addInTest2: 32796 gas

### Recommended Mitigation Steps
Use bytes32 for `mapping(string => Quest) public quests;`

# [05-GAS] Public Functions To External
The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.
TODO: има още
QuestFactory:
61: `function createQuest(.....)`
142: `function changeCreateQuestRole(address account_, bool canCreateQuest_)`
159: `function setClaimSignerAddress(address claimSignerAddress_)`
172: `function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_)`
179: `function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) `
219: `function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_)`
Erc20Quest:
102: `withdrawFee()`
RabbitHoleReceipt:
65: `function setReceiptRenderer(address receiptRenderer_)`
71: `function setRoyaltyRecipient(address royaltyRecipient_) `
77: `function setQuestFactory(address questFactory_) `
83: `setMinterAddress(address minterAddress_)`
90: `function setRoyaltyFee(uint256 royaltyFee_)`
RabbitHoleTickets:
54: `setTicketRenderer(address ticketRenderer_) `
60: ` setRoyaltyRecipient(address royaltyRecipient_)`
# [06-GAS] Using unchecked blocks to save gas - Increments in for loop can be unchecked
Quest:
70: `for (uint i = 0; i < tokenIds_.length; i++)` 
104: `for (uint i = 0; i < tokens.length; i++) {`
RabbitHoleReceipt:
117: `for (uint i = 0; i < msgSenderBalance; i++) {`
121: `foundTokens++;`
128 `for (uint i = 0; i < msgSenderBalance; i++)`
131: `filterTokensIndexTracker++;`

# [07-GAS] require() strings longer than 32 bytes cost extra gas
RabbitHoleReceipt
161: `require(_exists(tokenId_), 'ERC721URIStorage: URI query for nonexistent token');`