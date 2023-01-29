### 1st BUG
Cache Array Length Outside of Loop

#### Impact
Issue Information: 
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

Example
🤦 Bad:

for (uint256 i = 0; i < array.length; i++) {
    // invariant: array's length is not changed
}
🚀 Good:

uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // invariant: array's length is not changed
}

#### Findings:
```
quest-protocol/contracts/Quest.sol::101 => if (tokens.length == 0) revert NoTokensToClaim();
```
#### Tools used
Manual VS code audit

### 2nd BUG
Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: 
⚡️ Only valid for solidity versions <0.6.12 ⚡️

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.

Example
🤦 Bad:

bytes32 public constant GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
🚀 Good:

bytes32 public immutable GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");

#### Findings:
```
quest-protocol/contracts/QuestFactory.sol::17 => bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
quest-protocol/contracts/RabbitHoleReceipt.sol::119 => if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
```
#### Tools used
Manual VS code audit

### 3rd BUG
Long Revert Strings

#### Impact
Issue Information: 
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

Example
🤦 Bad:

require(condition, "UniswapV3: The reentrancy guard. A transaction cannot re-enter the pool mid-swap");
🚀 Good (with shorter string):

// TODO: Provide link to a reference of error codes
require(condition, "LOK");
🚀 Good (with custom errors):

/// @notice A transaction cannot re-enter the pool mid-swap.
error NoReentrancy();

// ...

if (!condition) {
    revert NoReentrancy();
}

#### Findings:
```
quest-protocol/contracts/ReceiptRenderer.sol::68 => '"description": "RabbitHole.gg Receipts are used to claim rewards from completed quests.",',
quest-protocol/contracts/ReceiptRenderer.sol::102 => '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
quest-protocol/contracts/ReceiptRenderer.sol::105 => '<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest #',
quest-protocol/contracts/ReceiptRenderer.sol::108 => '<text x="70%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #',
quest-protocol/contracts/TicketRenderer.sol::24 => '"description": "A reward for completing quests within RabbitHole, with unk(no)wn utility",',
quest-protocol/contracts/TicketRenderer.sol::38 => '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
quest-protocol/contracts/TicketRenderer.sol::41 => '<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Tickets #',
```
#### Tools used
Manual VS code audit

### 4th BUG
Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: 
A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

Example
🤦 Bad:

uint256 b = a / 2;
uint256 c = a / 4;
uint256 d = a * 8;
🚀 Good:

uint256 b = a >> 1;
uint256 c = a >> 2;
uint256 d = a << 3;

#### Findings:
```
quest-protocol/contracts/ReceiptRenderer.sol::102 => '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
quest-protocol/contracts/TicketRenderer.sol::38 => '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
```
#### Tools used
Manual VS code audit