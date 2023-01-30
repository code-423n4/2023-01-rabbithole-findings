# [01-NC] Remove forgotten TODO
The code that contains “open todos” reflects that the development is not finished and that the code can change a posteriori, prior release, with or without audit.
File IQuest.sol: [4](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L4)

# [02-NC] Use more recent version of solidity

# [03-NC] Mismatches between contracts and interfaces
Consider applying the necessary changes in the mentioned interfaces and contracts so that definitions and implementations fully match.

# [04-NC] Solidity compiler optimizations can be problematic
Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

### Recommended Mitigation Steps
Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug.Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.


# [05-NC] Constants are not being declared explicitly
Developers should define a constant variable for every magic value used (including booleans), giving it a clear and self-explanatory name. Additionally, for complex values, inline comments explaining how they were calculated or why they were chosen are highly recommended. Following Solidity’s style guide, constants should be named in UPPER_CASE_WITH_UNDERSCORES format, and specific public getters should be defined to read each one of them.
QuestFactory:
48: `setQuestFee(2_000);  // 2_000`
187: ` if (questFee_ > 10_000) revert QuestFeeTooHigh();  // 10_000`
Erc20Quest:
53: `return (maxTotalRewards() * questFee) / 10_000;`
97:  `return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;`
RabbitHoleReceipt:
184: `uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;`
### Recommended Mitigation Steps
`uint256 private constant MAXIMUM_FEE_BASIS_POINTS = 10_000;`
`uint256 private constant INITIAL_FEE_BASIS_POINTS = 2_000;`

# [06-NC] Argument naming and error can lead to confusion
QuestFactory:
68: `string memory questId_`
70: `if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();`

Naming of input argument and error can lead to confusion. In the contract has state variable `questIdCount` which is inceased when new quest is created.. Argument `questId_` and error `QuestIdUsed()` are more related to variable `mapping(string => Quest) public quests;` but with this naming give to us more information that variable `questIdCount` is involved or will be changes.

### Recommended Mitigation Steps
Rename argument variable and error
`questName_`
`QuestNameAlreadyUsed`

# [07-NC] Define enum struct for quest type
Currently in the protocol exist two quest type erc20 and erc1155. When new quest is created we are checking for quest type
QuestFactory:
72: `if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20')))`
105: `if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155')))`

When we are checking type of contract two times keccak256 and abi.encodePacked() is used. It will be more readability and cheap to use enum struct for that effort.
### Recommended Mitigation Steps
Use enum struct for contract type
```
enum QuestType {
  ERC20,
  ERC1155
}
if (contractType_ == QuestType.ERC20)
if (contractType_ == QuestType.ERC1155)
```

# [09-NC] Emit event when important storage variable is changed
It is good practice every time when important storage variable is changed to be emited event.
QuestFactory:
142: `changeCreateQuestRole(address account_, bool canCreateQuest_)`
159: `setClaimSignerAddress(address claimSignerAddress_) `
165: `setProtocolFeeRecipient(address protocolFeeRecipient_)`
172: `setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) `
179: `setRewardAllowlistAddress(address rewardAddress_, bool allowed_) `
186: `setQuestFee(uint256 questFee_) `
RabbitHoleReceipt:
65: `function setReceiptRenderer(address receiptRenderer_)`
71: `function setRoyaltyRecipient(address royaltyRecipient_) `
77: `function setQuestFactory(address questFactory_) `

Consider that this will increase gas and will be more expensive.

# [01-LR] Does not allow quest fee to be 100%
Currently, the quest fee is allowed to be from 0% to 100% included.
` if (questFee_ > 10_000)`
User may not like to see that owner can set fee to 100% when he wants or if owner is compromised the fee can be changed to 100%. 
Like UniSwap and other protocol is good to have some limit of setting of the fee percent which is lower than 100%.