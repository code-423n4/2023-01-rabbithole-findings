## [L] Ownable two step transfer

The protocol use openzeppelin ownable contract `import {OwnableUpgradeable} from '@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol';`. This contract doesn't have a two step transfer pattern.

Recommend considering implementing a two step process where the owner or admin nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

## [L] Missing `onlyAdmin` part (or wrong modifier name)

```solidity
File: Quest.sol
76:     modifier onlyAdminWithdrawAfterEnd() {
77:         if (block.timestamp < endTime) revert NoWithdrawDuringClaim();
78:         _;
79:     }
```

The `onlyAdmin` part is not defined in the modifier, it only check the time. Either the modifier name is wrong, might be `onlyWithdrawAfterEnd` since the only usage is on `withdrawRemainingTokens()` function which already include the `onlyOwner`, or indeed the modifier is missing the `onlyAdmin` part/implementation.

```solidity
File: Quest.sol
150:     function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
```

## [L] Inexistent Sanitization of Input Address

Zero address check. The linked address argument affects a sensitive contract variable yet remains unsanitized. We advise it to be sanitized against the zero-address (address(0)) to prevent misconfiguration of the contract.

```solidity
File: QuestFactory.sol
37:     function initialize(
38:         address claimSignerAddress_,
39:         address rabbitholeReceiptContract_,
40:         address protocolFeeRecipient_
41:     ) public initializer {

File: Quest.sol
40:         rewardToken = rewardTokenAddress_;
```

## [L] Inexistent Event Emissions

The linked functions adjust sensitive contract variables yet do not emit an event for them.
We advise an event to be coded and correspondingly emitted to ensure off-chain processes can properly react to these changes.

```solidity
File: RabbitHoleReceipt.sol
63:     /// @dev set the receipt renderer contract
64:     /// @param receiptRenderer_ the address of the receipt renderer contract
65:     function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
66:         ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);
67:     }
68:
69:     /// @dev set the royalty recipient
70:     /// @param royaltyRecipient_ the address of the royalty recipient
71:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
72:         royaltyRecipient = royaltyRecipient_;
73:     }
74:
75:     /// @dev set the quest factory contract
76:     /// @param questFactory_ the address of the quest factory contract
77:     function setQuestFactory(address questFactory_) public onlyOwner {
78:         QuestFactoryContract = IQuestFactory(questFactory_);
79:     }
```

More on QuestFactory.sol

## [N] Typos

- remave

## [N] Refactor

This two function can be packed into setPause(bool status) function, further more add a check require to prevent same state changes.

```solidity
File: Quest.sol
57:     function pause() public onlyOwner onlyStarted {
58:         isPaused = true;
59:     }
60:
61:     /// @notice Unpauses the Quest
62:     /// @dev Only the owner of the Quest can call this function. Also requires that the Quest has started (not by date, but by calling the start function)
63:     function unPause() public onlyOwner onlyStarted {
64:         isPaused = false;
65:     }
```

## [N] Standarize usage of `uint` vs `uint256`

There are some interchangeable type being use in the project, `uint` and `uint256`, even thought it's the same meaning, but it's better to keep in standard practice, only use one.
