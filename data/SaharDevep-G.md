# Audit Report

## Summery
[G01] Not using the named return variables when a function returns, wastes deployment gas
[G02] Use custom errors rather than revert()/require() strings to save gas
[G03] Pack the state variables

## issues Found

### [G01] Not using the named return variables when a function returns, wastes deployment gas

#### Fidings:
```
quest-protocol\contracts\RabbitHoleReceipt.sol::178 => function royaltyInfo(
quest-protocol\contracts\RabbitHoleTickets.sol::109 => function royaltyInfo(
```

#### Tools used
Manual

### [G02] Use custom errors rather than revert()/require() strings to save gas

#### Fidings:
```
quest-protocol\contracts\RabbitHoleReceipt.sol::161 => require(_exists(tokenId_), 'ERC721URIStorage: URI query for nonexistent token');
quest-protocol\contracts\RabbitHoleReceipt.sol::162 => require(QuestFactoryContract != IQuestFactory(address(0)), 'QuestFactory not set');
quest-protocol\contracts\RabbitHoleReceipt.sol::182 => require(_exists(tokenId_), 'Nonexistent token'); 
```

#### Tools used
Manual

### [G03] Pack the state variables
Turn this:
```
    RabbitHoleReceipt public immutable rabbitHoleReceiptContract;
    address public immutable rewardToken;
    uint256 public immutable endTime;
    uint256 public immutable startTime;
    uint256 public immutable totalParticipants;
    uint256 public immutable rewardAmountInWeiOrTokenId;
    bool public hasStarted;
    bool public isPaused;
    string public questId;
    uint256 public redeemedTokens;
```

To this:
```
    RabbitHoleReceipt public immutable rabbitHoleReceiptContract;
    address public immutable rewardToken;
    uint256 public immutable endTime;
    uint256 public immutable startTime;
    uint256 public immutable totalParticipants;
    uint256 public immutable rewardAmountInWeiOrTokenId;
    uint256 public redeemedTokens; // Moved up
    bool public hasStarted;
    bool public isPaused;
    string public questId;
```


#### Findings:
```
quest-protocol\contracts\Quest.sol::22 => uint256 public redeemedTokens;
```

#### Tools used
Manual
