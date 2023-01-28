## [YO GAS-1] USING BOOLS FOR STORAGE INCURS OVERHEAD

### Handle
yosuke

## Vulnerability details
### Impact
Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and  pointer aliasing, and it cannot be disabled.

### Proof of Concept
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L135
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L168
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L192
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L121
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L25
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L44


### Recommended Mitigation Steps
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past.
ex)
before
```solidity=
bool claimed_,
```
after
```solidity=
uint256 claimed_,
```

## [YO GAS-2] Using private rather than public for constants and immutable, saves gas

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L13-L15
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L13-L18

### Recommended Mitigation Steps
ex)
before
```solidity=
uint256 public immutable questFee;
```
after
```solidity=
uint256 private immutable questFee;
```

## [YO GAS-3] USE bytes32 INSTEAD OF string

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L23
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L19
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L21
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L32
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L28
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L30
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L98
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L110
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L160
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L164
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L102
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L28
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L42
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L82
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L100
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L18
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L36


### Recommended Mitigation Steps
USE bytes32 INSTEAD OF string because of lower gas fee.
ex)
before

```solidity=
string memory questId_,
```

after
```solidity=
bytes32 memory questId_,
```
## [YO GAS-4] `++I`/`I++` SHOULD BE `UNCHECKED{++I}`/`UNCHECKED{I++}` WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN `FOR`AND `WHILE`LOOPS

### Handle
yosuke

## Vulnerability details
### Impact

The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**.

### Proof of Concept
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L104
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L128


### Recommended Mitigation Steps

```solidity
for (uint256 i = 0; i < length; i++) {
	...
	**UNCHECKED{I++}**
}
```
## [YO GAS-5]**`X = X + Y`, `X = X - Y`, `X = X * Y`, `X = X / Y` ARE MORE EFFICIENT, THAN `X += Y`, `X -= Y`, `X *= Y`, `X /= Y`**

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115


### Recommended Mitigation Steps
ex)
before
```solidity=
redeemedTokens += redeemableTokenCount;
```
after
```solidity=
redeemedTokens = redeemedTokens + redeemableTokenCount;
```

## [YO GAS-6] USE FUNCTION INSTEAD OF MODIFIERS

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L58
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L76
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L82
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L88

### Recommended Mitigation Steps
USE FUNCTION INSTEAD OF MODIFIERS because of lower gas fee.
ex)
before
```solidity=
modifier onlyAdminWithdrawAfterEnd() {
    if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
    _;
}

```
after
```solidity=
function onlyAdminWithdrawAfterEnd() private {
    if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
}

```

## [YO GAS-7] USE CUSTOM ERRORS RATHER THAN REVERT()/REQUIRE() STRINGS TO SAVE GAS

### Handle
yosuke

## Vulnerability details
### Impact
Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they’re hit by avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas.

### Proof of Concept
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L161
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L162
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L182

### Recommended Mitigation Steps
USE CUSTOM ERRORS RATHER THAN REVERT()/REQUIRE() STRINGS TO SAVE GAS

## [YO GAS-8] Gas savings by moving bool state variable next to an address state variable declaration

### Handle
yosuke

## Vulnerability details
### Impact
A bool in Solidity is internally represented as a unit8 and so required only 8 bits of the 256-bits storage slot. An address variable is 160-bits. So declaring a bool next to an address variable lets Solidity pack them in the same storage slot thereby using one slot instead of two.

For reference, see https://mudit.blog/solidity-gas-optimization-tips/

### Proof of Concept
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L14-L20
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L25-L27
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L44-L46

### Recommended Mitigation Steps
Move bool state variable declaration next to an address state variable declaration.
ex)
before
```solidity=
address public immutable rewardToken;
uint256 public immutable endTime;
uint256 public immutable startTime;
uint256 public immutable totalParticipants;
uint256 public immutable rewardAmountInWeiOrTokenId;
bool public hasStarted;
bool public isPaused;
```
after
```solidity=
bool public hasStarted;
bool public isPaused;
address public immutable rewardToken;
uint256 public immutable endTime;
uint256 public immutable startTime;
uint256 public immutable totalParticipants;
uint256 public immutable rewardAmountInWeiOrTokenId;
```