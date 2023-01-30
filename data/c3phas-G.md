### Table of contents
- [FINDINGS](#findings)
- [Using immutable on variables that are only set in the constructor and never after](#using-immutable-on-variables-that-are-only-set-in-the-constructor-and-never-after)
- [Internal/Private functions only called once can be inlined to save gas](#internalprivate-functions-only-called-once-can-be-inlined-to-save-gas)
- [require() or revert() statements that check input arguments should be at the top of the function](#require-or-revert-statements-that-check-input-arguments-should-be-at-the-top-of-the-function)
  - [Reorder the if's to save gas on early reverts](#reorder-the-ifs-to-save-gas-on-early-reverts)
- [Multiple accesses of a mapping/array should use a local variable cache](#multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache)
  - [QuestFactory.sol.createQuest(): quests\[questId\_\] should be cached in local storage](#questfactorysolcreatequest-questsquestid_-should-be-cached-in-local-storage)
  - [QuestFactory.sol.createQuest(): quests\[questId\_\] should be cached in local storage](#questfactorysolcreatequest-questsquestid_-should-be-cached-in-local-storage-1)
  - [QuestFactory.sol.mintReceipt(): quests\[questId\_\] should be cachec in local storage](#questfactorysolmintreceipt-questsquestid_-should-be-cachec-in-local-storage)
- [Using storage instead of memory for structs/arrays saves gas](#using-storage-instead-of-memory-for-structsarrays-saves-gas)
- [x += y costs more gas than x = x + y for state variables](#x--y-costs-more-gas-than-x--x--y-for-state-variables)
- [Using unchecked blocks to save gas - Increments in for loop can be unchecked  ( save 30-40 gas per loop iteration)](#using-unchecked-blocks-to-save-gas---increments-in-for-loop-can-be-unchecked---save-30-40-gas-per-loop-iteration)
- [Boolean comparisons](#boolean-comparisons)
- [A modifier used only once and not being inherited should be inlined to save gas](#a-modifier-used-only-once-and-not-being-inherited-should-be-inlined-to-save-gas)

## FINDINGS
NB: Some functions have been truncated where necessary to just show affected parts of the code
Through out the report some places might be denoted with audit tags to show the actual place affected.

## Using immutable on variables that are only set in the constructor and never after 
Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas)

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

For short strings you can simulate immutable via a function that returns the value or you can store into a bytes32 for the majority of short strings.


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L21
**Gas saved: 1 Instance = 2.1k**
`Total Instances: 1, total gas: 1* 2.1k= 2.1k`

```solidity
File: /contracts/Quest.sol
21:    string public questId;
```

## Internal/Private functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

Affected code:
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L152
```solidity
File: /contracts/QuestFactory.sol
152:    function grantDefaultAdminAndCreateQuestRole(address account_) internal {
```
the above function is only called on [Line 44](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L44)


##  require() or revert() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

### Reorder the if's to save gas on early reverts
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L88-L92
```solidity
File: /contracts/Quest.sol
88:    modifier onlyQuestActive() {
89:        if (!hasStarted) revert NotStarted();
90:        if (block.timestamp < startTime) revert ClaimWindowNotStarted();
91:        _;
92:    }
```

It would be expensive to read `hasStarted` as it involves an SLOAD. If `if (block.timestamp < startTime) ` the check fails, we would have wasted gas computing `!hasStarted`. Thus it's better to fail cheaply by reordering the if's as follows

```diff
diff --git a/contracts/Quest.sol b/contracts/Quest.sol
index 33163ee..8a17acc 100644
--- a/contracts/Quest.sol
+++ b/contracts/Quest.sol
@@ -86,8 +86,8 @@ contract Quest is Ownable, IQuest {

     /// @notice Checks if quest has started both at the function level and at the start time
     modifier onlyQuestActive() {
-        if (!hasStarted) revert NotStarted();
         if (block.timestamp < startTime) revert ClaimWindowNotStarted();
+        if (!hasStarted) revert NotStarted();
         _;
     }

```

## Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time.
**Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. 
As an example, instead of repeatedly calling ```someMap[someIndex]```, save its reference like this: ```SomeStruct storage someStruct = someMap[someIndex]``` and use it.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61-L103
### QuestFactory.sol.createQuest(): quests[questId_] should be cached in local storage
```solidity
File: /contracts/QuestFactory.sol
61:    function createQuest(

69:    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
70:        if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();//@audit: 1st access

98:            quests[questId_].questAddress = address(newQuest);//@audit:2nd access
99:            quests[questId_].totalParticipants = totalParticipants_;//@audit:3rd access
100:            newQuest.transferOwnership(msg.sender);
101:            ++questIdCount;
102:            return address(newQuest);
103:        }
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L129-L130
### QuestFactory.sol.createQuest(): quests[questId_] should be cached in local storage
```solidity
File: /contracts/QuestFactory.sol
129:            quests[questId_].questAddress = address(newQuest);//@audit: accessed here
130:            quests[questId_].totalParticipants = totalParticipants_;//@audit:accessed here
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219-L229
### QuestFactory.sol.mintReceipt(): quests[questId_] should be cachec in local storage
```solidity
File: /contracts/QuestFactory.sol
219:    function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
220:        if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
221:        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();

225:        quests[questId_].addressMinted[msg.sender] = true;
226:        quests[questId_].numberMinted++;
227:        emit ReceiptMinted(msg.sender, questId_);
228:        rabbitholeReceiptContract.mint(msg.sender, questId_);
229:    }
```


## Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L164
```solidity
File: /contracts/RabbitHoleReceipt.sol
164:        string memory questId = questIdForTokenId[tokenId_];
```

```diff
diff --git a/contracts/RabbitHoleReceipt.sol b/contracts/RabbitHoleReceipt.sol
index 085b617..79061ea 100644
--- a/contracts/RabbitHoleReceipt.sol
+++ b/contracts/RabbitHoleReceipt.sol

-        string memory questId = questIdForTokenId[tokenId_];
+        string storage questId = questIdForTokenId[tokenId_];
```

## x += y costs more gas than x = x + y for state variables

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115
```solidity
File: /contracts/Quest.sol
115:        redeemedTokens += redeemableTokenCount;
```

```diff
diff --git a/contracts/Quest.sol b/contracts/Quest.sol
index 33163ee..a848e48 100644
--- a/contracts/Quest.sol
+++ b/contracts/Quest.sol
@@ -112,7 +112,7 @@ contract Quest is Ownable, IQuest {
         uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);
         _setClaimed(tokens);
         _transferRewards(totalRedeemableRewards);
-        redeemedTokens += redeemableTokenCount;
+        redeemedTokens = redeemedTokens + redeemableTokenCount;

         emit Claimed(msg.sender, totalRedeemableRewards);
     }
```

## Using unchecked blocks to save gas - Increments in for loop can be unchecked  ( save 30-40 gas per loop iteration)
The majority of Solidity for loops increment a uint256 variable that starts at 0. These increment operations never need to be checked for over/underflow because the variable will never reach the max number of uint256 (will run out of gas long before that happens). The default over/underflow check wastes gas in every iteration of virtually every for loop . eg.


**Affected code**
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117
```solidity
File: /contracts/RabbitHoleReceipt.sol

117:        for (uint i = 0; i < msgSenderBalance; i++) {

128:        for (uint i = 0; i < msgSenderBalance; i++) {
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70
```solidity
File: /contracts/Quest.sol
70:        for (uint i = 0; i < tokenIds_.length; i++) {

104:        for (uint i = 0; i < tokens.length; i++) {
```

## Boolean comparisons

Comparing to a constant (true or false) is a bit more expensive than directly checking the returned boolean value.
I suggest using if(directValue) instead of if(directValue == true) and if(!directValue) instead of if(directValue == false) here:

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73
```solidity
File: /contracts/QuestFactory.sol
73:            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L136
```solidity
File: /contracts/Quest.sol
136:        return claimedList[tokenId_] == true;
```

## A modifier used only once and not being inherited should be inlined to save gas
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L58-L61
```solidity
File: /contracts/RabbitHoleReceipt.sol
58:    modifier onlyMinter() {
59:        msg.sender == minterAddress;
60:        _;
61:    }
```
The above modifier is only used on the following function 
```solidity
File: /contracts/RabbitHoleReceipt.sol
98:    function mint(address to_, string memory questId_) public onlyMinter {
99:        _tokenIds.increment();
100:        uint newTokenID = _tokenIds.current();
101:        questIdForTokenId[newTokenID] = questId_;
102:        timestampForTokenId[newTokenID] = block.timestamp;
103:        _safeMint(to_, newTokenID);
104:    }
```

We could modify it as follows

```diff
diff --git a/contracts/RabbitHoleReceipt.sol b/contracts/RabbitHoleReceipt.sol
index 085b617..c834002 100644
--- a/contracts/RabbitHoleReceipt.sol
+++ b/contracts/RabbitHoleReceipt.sol
@@ -55,10 +55,6 @@ contract RabbitHoleReceipt is
         ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);
     }

-    modifier onlyMinter() {
-        msg.sender == minterAddress;
-        _;
-    }

     /// @dev set the receipt renderer contract
     /// @param receiptRenderer_ the address of the receipt renderer contract
@@ -95,7 +91,8 @@ contract RabbitHoleReceipt is
     /// @dev mint a receipt
     /// @param to_ the address to mint to
     /// @param questId_ the quest id
-    function mint(address to_, string memory questId_) public onlyMinter {
+    function mint(address to_, string memory questId_) public {
+        require( msg.sender == minterAddress);
         _tokenIds.increment();
         uint newTokenID = _tokenIds.current();
         questIdForTokenId[newTokenID] = questId_;
```


https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L88-L92
```solidity
File: /contracts/Quest.sol
88:    modifier onlyQuestActive() {
89:        if (!hasStarted) revert NotStarted();
90:        if (block.timestamp < startTime) revert ClaimWindowNotStarted();
91:        _;
92:    }
```
The above modifier is only called on [Line 96](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96)
