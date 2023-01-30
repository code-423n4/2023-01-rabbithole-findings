# Table of contents

- [L-01] Use safeTransferOwnership instead of transferOwnership function
- [L-02] Test coverage is only at 89%, where it should be 100%
- [L-03] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY
- [L-04] LOSS OF PRECISION DUE TO ROUNDING
- [L-05] LACK OF CHECKS ADDRESS(0)
- [N-01] To lock pragmas to specific compiler version
- [N-02] Should use bytes.concat() instead of abi.encodePacked
- [N-03] Rename nonClaimableTokens variable 
- [N-04] INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS
- [N-05] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES
- [S-01] Generating Perfect Code Headers

## [L-01] Use safeTransferOwnership instead of transferOwnership function

Description:
transferOwnership function is used to change Ownership from Owned.sol.

Use a 2 structure transferOwnership which is safer.
safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.

##  [L-02] Test coverage is only at 89%, where it should be 100%

It’s crucial to write tests with possibly 100% coverage for smart contract systems.
It is recommended to write appropriate tests for all possible code streams and especially for extreme cases.
But the other important point is the test context.
Tests written with Solidity are safer, so it is recommended to focus on tests with Foundry.


## [L-03] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY
initialize() function can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the __Ownable_init() function. Also, there is no 0 address check in the address arguments of the initialize() function, which must be defined.
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37-L50

Recommended Mitigation Steps
Add a control that makes initialize() only call the Deployer Contract;

## [L-04] LOSS OF PRECISION DUE TO ROUNDING
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L95-L98

## [L-05] LACK OF CHECKS ADDRESS(0)
The following methods have a lack of checks if the received argument is an address, it’s good practice in order to reduce human error to check that the address specified in the constructor or initialize is different than address(0).

Affected Code: 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71-L73
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L83-L86
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L179
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L40
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L44


## [N-01] To lock pragmas to specific compiler version

Currently: pragma solidity ^0.8.15 was used across all files
Issue: Should not use ^ due as some functions may be deprecate and will not be able to function as it is

Description:
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
https://swcregistry.io/docs/SWC-103

Recommendation:
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
solidity-specific/locking-pragmas

Affected Code:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L2
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L2
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L2
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L2

## [N-02]  Should use bytes.concat() instead of abi.encodePacked
Affected Code: 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L105

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled.
Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)

## [N-03] Rename nonClaimableTokens variable 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L85
Suggest to change a more understandable/readable variable name.

## [N-04] INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L133-L137
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L139-L142
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L144-L147
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L157-L173
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L175-L186
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L188-L194
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L191-L195
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L197-L205
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L207-L213
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L95-L98

Context
All Contracts
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
If Return parameters are declared, you must prefix them with ”/// @return”.
Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.

Recommended Mitigation Steps
Include return parameters in NatSpec comments

## [N-05] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

Description:
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

Affected Code:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L4-L13

Example:
import {contract1 , contract2} from "RabbitHoleReceipt.sol";

## [S-01] Generating Perfect Code Headers
I would recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/


