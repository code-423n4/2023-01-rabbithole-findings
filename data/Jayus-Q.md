1. Protocol fees should be less than 10000 bps to avoid charging 100% fees.
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L187

2. Use recommended EIP712 for signature verification as it is more secure.

3. Use bytes32 for `questId` by taking the keccack256 hash of the string input.

4. Do not mix `uint` and `uint256`. Use one throughout for uniformity, preferably `uint256`.