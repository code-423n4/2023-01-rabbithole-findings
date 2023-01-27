# 1. QuestFactory.sol: Quest struct can be better packed
The Quest struct has empty spaces in storage slots.
```solidity
struct Quest {
        mapping(address => bool) addressMinted; // mapping 
        address questAddress; // slot 1
        uint totalParticipants; // slot 2
        uint numberMinted; // slot 3
    }
```
However uint256 isn't necessary for storing number of participants or numberMinted, since both will be much much lower than 2^256. The 160 bit address leaves 96 empty bits, whach can be used by a uint96 to save a storage slot. The struct can even be packed all in one slot by using uint48, which can store a max value of 2^48=2.8e14, which might be enough for most cases.

```solidity
struct Quest {
        mapping(address => bool) addressMinted; // mapping 
        address questAddress; // slot 1
        uint48 totalParticipants; // slot 1
        uint48 numberMinted; // slot 1
    }
```

# 2. Quest.sol: l13-l22- public variables can be better packed
Addresses (including contract objects) take up 160 bits. the remaining 96 bits can be used to store uint96 data saving storage slots and gas.
Original:
```solidity
    RabbitHoleReceipt public immutable rabbitHoleReceiptContract; // slot 1
    address public immutable rewardToken; // slot 2
    uint256 public immutable endTime; // slot 3
    uint256 public immutable startTime; // slot 4
    uint256 public immutable totalParticipants; // slot 5
    uint256 public immutable rewardAmountInWeiOrTokenId; // slot 6
    bool public hasStarted; // slot 7
    bool public isPaused; // slot 7
    uint256 public redeemedTokens; // slot 8
```
Packed. timestamps fit well in uint96, 
```solidity
RabbitHoleReceipt public immutable rabbitHoleReceiptContract; // slot 1 (160 bits)
uint96 public immutable startTime; // slot 1 (96bits)

address public immutable rewardToken; // slot 2
uint96 public immutable endTime; // slot 2

uint120 public immutable totalParticipants; // slot 3
uint120 public redeemedTokens; // slot 3
bool public hasStarted; // slot 3
bool public isPaused; // slot 3

uint256 public immutable rewardAmountInWeiOrTokenId; // slot 4
```
This saves 4 slots per Quest.