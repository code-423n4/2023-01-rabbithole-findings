# QA Report (low/non-critical)

## Changing `claimSignerAddress` invalidates existing hashes/signatures

In `QuestFactory.sol`, `setClaimSignerAddress` allows the owner to change the `claimSignerAddress`, however doing so will break `mintReceipt` functionality for any existing hashes/signatures that have not yet been used to mint a receipt.

### Link to code

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219-L229