# Table of contents

- [L-01] Use safeTransferOwnership instead of transferOwnership function
- [L-02] Test coverage is only at 89%, where it should be 100%
- [N-01] pragma solidity ^ should not be used
- [N-02] abi encode
- [N-03] Rename nonClaimableTokens variable 
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


## [N-01] pragma solidity ^ should not be used

Currently: pragma solidity ^0.8.15 was used across all files

Solution: Should not use ^ due as some functions may be deprecate and will not be able to function as it is

## [N-02] abi encode

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L105

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled.
Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)

## [N-03] Rename nonClaimableTokens variable 
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L85
Suggest to change a more understandable/readable variable name.

## [S-02] Generating Perfect Code Headers
I would recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/


