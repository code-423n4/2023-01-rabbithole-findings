### Unspecific Compiler Version Pragma

#### Impact
Issue Information: 
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

Example
🤦 Bad:

pragma solidity ^0.8.0;
🚀 Good:

pragma solidity 0.8.4;

#### Findings:
```
quest-protocol/contracts/Erc1155Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/Erc20Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/QuestFactory.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/RabbitHoleReceipt.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/RabbitHoleTickets.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/ReceiptRenderer.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/TicketRenderer.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/interfaces/IQuest.sol::2 => pragma solidity ^0.8.15;
quest-protocol/contracts/interfaces/IQuestFactory.sol::2 => pragma solidity ^0.8.15;
```
#### Tools used
Manual VS code audit