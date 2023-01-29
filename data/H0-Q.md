## [N-01] Use latest Solidity Version

Context:
All contracts

Description:
For security, it is best practice to use the latest Solidity version.
For the security fix list in the versions;
(https://github.com/ethereum/solidity/blob/develop/Changelog.md)

Recommendation:
Old version of Solidity is used , newer version can be used (0.8.17)

## [N-02] FOR FUNCTIONS, FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS
[QuestFactory.sol line 152](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152)
The above codes don’t follow Solidity’s standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)
