## [L-01] OWNER CAN RENOUNCE OWNERSHIP
### Description
Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

Ownable used in this project contract implements renounceOwnership . This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

onlyOwner functions;
 [QuestFactory.sol#L9](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L9) 
[RabbitHoleReceipt.sol#L7](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L7)
[RabbitHoleTickets.sol#L5](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L5)   
### Recommended Mitigation Steps**
We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.
   
## [L-02] USE OF FLOATING PRAGMA    
### Description:
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
https://swcregistry.io/docs/SWC-103
    
### Context:
[RabbitHoleTickets.sol#L2 ](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L2)   
[QuestFactory.sol#L2](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L2)
[RabbitHoleReceipt.sol#L2](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L2)
[Quest.sol#L2](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L2)
[ReceiptRenderer.sol#L2](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L2)
[Erc1155Quest.sol#L2](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L2)
[Erc20Quest.sol#L2](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L2)
[TicketRenderer.sol#L2](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L2)   
[IQuest.sol#L2](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L2)
[IQuestFactory.sol#L2](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L2)

### Recommendation:
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
solidity-specific/locking-pragmas
## [S-01] GENERATE PERFECT CODE HEADERS EVERY TIME
### Description
I recommend using header for Solidity code layout and readability.

https://github.com/transmissions11/headers

    