## Use of named returns for local variables saves gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:

[File: QuestFactory.sol#L61-L103](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61-L103)

```diff
    function createQuest(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountOrTokenId_,
        string memory contractType_,
        string memory questId_
-    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
+    ) public onlyRole(CREATE_QUEST_ROLE) returns (address newQuestAddr) {

            [ ... ]

-            return address(newQuest);
+            newQuestAddr = address(newQuest);
```
## Avoid comparing boolean expressions to boolean literals
Comparing a boolean value to a boolean literal incurs the `ISZERO` operation and costs more gas than using a boolean expression.

Here are some of the affected code lines that may be refactored as follows:

[File: QuestFactory.sol#L73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73)

```diff
-            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
+            if (!rewardAllowlist[rewardTokenAddress_]) revert RewardNotAllowed();
```
[File: QuestFactory.sol#L221](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221)

```diff
-        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
+        if (quests[questId_].addressMinted[msg.sender]) revert AddressAlreadyMinted();
```
## Non-strict inequalities are cheaper than strict ones
In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).

As an example, consider replacing `<=` with the strict counterpart `<` in the following inequality instance:

[File: Quest.sol#L35-L37](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L35-L37)

```diff
-        if (endTime_ <= block.timestamp) revert EndTimeInPast();
// Rationale for adding 1 on the right side of the inequality:
// x <= 10; [10, 9, 8, ...]
// x < 10 + 1 is the same as x < 11; [10, 9, 8 ...]
+        if (endTime_ < block.timestamp + 1) revert EndTimeInPast();
```
## Merging of two opposing functions into a toggling function
`pause()` and `unpause()` in Quest.sol are two opposing functions that can be merged into a toggling function to save gas on contract deployment.

[File: Quest.sol#L57-L65](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57-L65)

```diff
-    function pause() public onlyOwner onlyStarted {
-        isPaused = true;
-    }

-    function unPause() public onlyOwner onlyStarted {
-        isPaused = false;
-    }

+    function pauser(bool status_) public onlyOwner onlyStarted {
+        isPaused = status_;
+    }
```
## Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A private visibility that saves more gas on function calls than the internal visibility is adopted because the modifier will only be making this call inside the contract.

For instance, the modifier instance below may be refactored as follows:

[File: Quest.sol#L88-L92](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L88-L92)

```diff
+    function _onlyQuestActive() private view {
+        if (!hasStarted) revert NotStarted();
+        if (block.timestamp < startTime) revert ClaimWindowNotStarted();
+    }

    modifier onlyQuestActive() {
-        if (!hasStarted) revert NotStarted();
-        if (block.timestamp < startTime) revert ClaimWindowNotStarted();
+        _onlyQuestActive();
        _;
    }
```
## Use storage instead of memory for structs/arrays
A storage pointer is cheaper since copying a state struct in memory would incur as many SLOADs and MSTOREs as there are slots. In another words, this causes all fields of the struct/array to be read from storage, incurring a Gcoldsload (2100 gas) for each field of the struct/array, and then further incurring an additional MLOAD rather than a cheap stack read. As such, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables will be much cheaper, involving only Gcoldsload for all associated field reads. Read the whole struct/array into a memory variable only when it is being returned by the function, passed into a function that requires memory, or if the array/struct is being read from another memory array/struct.

Here are some of the instances entailed:

[File: RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol)

```solidity
114:        uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);

125:        uint[] memory filteredTokens = new uint[](foundTokens);
```
## Redundant modifier visibility
`withdrawRemainingTokens()` in Erc20Quest.sol and Erc1155Quest.sol has `super.withdrawRemainingTokens()` execute in the first code line of the function logic. Since the parental function will have the same modifier `onlyOwner` run again, the child function may have the same modifier removed from its visibility (just as it has been done on [`start()`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L32-L37)) to save gas both on contract deployment and function calls:

[File: Erc20Quest.sol#L81-L82](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81-L82)
[File: Erc1155Quest.sol#L54-L55](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54-L55)

```diff
-    function withdrawRemainingTokens(address to_) public override onlyOwner {
+    function withdrawRemainingTokens(address to_) public override {
        super.withdrawRemainingTokens(to_);

        [ ... ]
```
## += and -= cost more gas
`+=` and `-=` generally cost 22 more gas than writing out the assigned equation explicitly. The amount of gas wasted can be quite sizable when repeatedly operated in a loop.

For instance, the `+=` instance below may be refactored as follows:

[File: Quest.sol#L115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115)

```diff
-        redeemedTokens += redeemableTokenCount;
+        redeemedTokens = redeemedTokens + redeemableTokenCount;
```
## Function order affects gas consumption
The order of function will also have an impact on gas consumption. Because in smart contracts, there is a difference in the order of the functions. Each position will have an extra 22 gas. The order is dependent on method ID. So, if you rename the frequently accessed function to more early method ID, you can save gas cost. Please visit the following site for further information:

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.15",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.