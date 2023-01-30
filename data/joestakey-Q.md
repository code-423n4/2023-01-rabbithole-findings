## Summary
### Low Risk
|      | Issue                                                                            |
|------|----------------------------------------------------------------------------------|
| L-01 | Additional sanity checks                                                         |
| L-02 | Wrong comment                                                                    |


### Non-Critical
|      | Issue                                                                            |
|------|----------------------------------------------------------------------------------|
| N-01 | Redundant computation                                                            |
| N-02 | Scientific notation can be used                                                  |
| N-03 | Use named imports                                                                |
| N-04 | Offset can be misleading                                                         |
| N-05 | Open TODOs                                                                       |
| N-06 | Events should be emitted in key setters                                          |
| N-07 | Avoid floating pragmas                                                           |


## Low

### [L‑01] Additional sanity checks


Add a sanity check in `setRoyaltyFee`: `royaltyFee_ < 10000`.

RabbitHoleReceipt.sol
```solidity
93: function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
94:         royaltyFee = royaltyFee_;
95:         emit RoyaltyFeeSet(royaltyFee_);
96:     }
```


### [L‑02] Wrong comment

RabbitHoleReceipt.sol
```solidity
177: /// @dev See {IERC165-royaltyInfo} //@audit - IERC2981
178:     /// @param tokenId_ the token id
179:     /// @param salePrice_ the sale price
180:     function royaltyInfo(
```

## Non-critical
### [N-01] Redundant computation  

`isPaused` is already false when `start` is called.
Quest.sol
```solidity
50: function start() public virtual onlyOwner {
51:         isPaused = false; //@audit can be removed
52:         hasStarted = true;
53:     }
```


### [N-02] Scientific notation can be used 


QuestFactory.sol
```solidity
186: function setQuestFee(uint256 questFee_) public onlyOwner {
187:         if (questFee_ > 10_000) revert QuestFeeTooHigh(); //@audit - change to 1e4
188:         questFee = questFee_;
189:     }
```

### [N-03] Use named imports


QuestFactory.sol
```solidity
10: import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol'; //@audit import {ECDSAUpgradeable}
```

### [N-04] Offset can be misleading

`questIdCount` should not be initialized to `1` in `initialize`, as it means querying this variable will always return the actual number of quests with an offset of 1.

QuestFactory.sol
```solidity
49:         questIdCount = 1; //@audit consider removing this line
```

### [N-05] Open TODOs

IQuest.sol
```solidity
4: // TODO clean this whole thing up
```

### [N-06] Events should be emitted in key setters

Emit an event in key setters

QuestFactory.sol
```solidity
159: function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
160:         claimSignerAddress = claimSignerAddress_;
161:     }
```

### [N-07] Avoid floating pragmas

All the contracts in scope
```solidity
2: pragma solidity ^0.8.15;
```
