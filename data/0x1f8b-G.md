- [Gas](#gas)
    - [**1. Avoid concat string constants**](#1-avoid-concat-string-constants)
        - [Total gas saved:](#total-gas-saved)
    - [**2. Optimize createQuest**](#2-optimize-createquest)
        - [Total gas saved:](#total-gas-saved)
    - [**3. Optimize mintReceipt**](#3-optimize-mintreceipt)
        - [Total gas saved:](#total-gas-saved)
    - [**4. Optimize getOwnedTokenIdsOfQuest**](#4-optimize-getownedtokenidsofquest)
    - [**5. Reduce boolean comparison**](#5-reduce-boolean-comparison)
        - [Total gas saved:](#total-gas-saved)
    - [**6. Avoid compound assignment operator in state variables**](#6-avoid-compound-assignment-operator-in-state-variables)
        - [Total gas saved:](#total-gas-saved)

# Gas

## **1. Avoid concat string constants**

Bundling consecutive strings together to prevent `encodePacked` from doing so can save gas

```diff
        bytes memory svg = abi.encodePacked(
+           '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350"><style>.base { fill: white; font-family: serif; font-size: 14px; }</style><rect width="100%" height="100%" fill="black" /><text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Tickets #',
-           '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
-           '<style>.base { fill: white; font-family: serif; font-size: 14px; }</style>',
-           '<rect width="100%" height="100%" fill="black" />',
-           '<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Tickets #',
            tokenId_.toString(),
+           '</text></svg>'
-           '</text>',
-           '</svg>'
        );
```

**Affected source code:**

- [ReceiptRenderer.sol:67-69](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L67-L69)
- [ReceiptRenderer.sol:84-91](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L84-L91)
- [ReceiptRenderer.sol:102-111](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L102-L111)
- [TicketRenderer.sol:19](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L19)
- [TicketRenderer.sol:30](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L30)
- [TicketRenderer.sol:37](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L37)
- [TicketRenderer.sol:46](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L46)

### Total gas saved:

In red the previous version, y green the new one:

```diff
- |  RabbitHoleReceipt                            ·          -  ·          -  ·     2776002  ·        [90m9.3 %[39m  ·          [32m[90m-[32m[39m  │
+ |  RabbitHoleReceipt                            ·          -  ·          -  ·     2775990  ·        [90m9.3 %[39m  ·          [32m[90m-[32m[39m  │
 ················································|·············|·············|··············|···············|··············
- |  ReceiptRenderer                              ·          -  ·          -  ·     1186932  ·          [90m4 %[39m  ·          [32m[90m-[32m[39m  │
+ |  ReceiptRenderer                              ·          -  ·          -  ·     1123645  ·        [90m3.7 %[39m  ·          [32m[90m-[32m[39m  │
```

## **2. Optimize `createQuest`**

It is possible to optimize the gas cost of the `createQuest` method by avoiding the `keccak256` computation.

```diff
+   bytes32 private constant ERC20_HASH = keccak256(abi.encodePacked('erc20'));
+   bytes32 private constant ERC1155_HASH = keccak256(abi.encodePacked('erc1155'));
    function createQuest(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountOrTokenId_,
        string memory contractType_,
        string memory questId_
    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
        if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();

-       if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
+       if (keccak256(abi.encodePacked(contractType_)) == ERC20_HASH) {
            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
        ...

-       if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
+       if (keccak256(abi.encodePacked(contractType_)) == ERC1155_HASH) {
            if (msg.sender != owner()) revert OnlyOwnerCanCreate1155Quest();
```

**Affected source code:**

- [QuestFactory.sol:72](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72)
- [QuestFactory.sol:105](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L105)

### Total gas saved:

In red the previous version, y green the new one:

```diff
- |  QuestFactory                                 ·          -  ·          -  ·     5313516  ·       [90m17.7 %[39m  ·          [32m[90m-[32m[39m  │
+ |  QuestFactory                                 ·          -  ·          -  ·     5313504  ·       [90m17.7 %[39m  ·          [32m[90m-[32m[39m  │
```

## **3. Optimize `mintReceipt`**

It is possible to optimize the gas cost of the `mintReceipt` method by avoiding multiple storage accesses.

```diff
    function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
+       Quest storage quest = quests[questId_];
-       if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
-       if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
+       if (quest.numberMinted + 1 > quest.totalParticipants) revert OverMaxAllowedToMint();
+       if (quest.addressMinted[msg.sender]) revert AddressAlreadyMinted();
        if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
        if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();

-       quests[questId_].addressMinted[msg.sender] = true;
-       quests[questId_].numberMinted++;
+       quest.addressMinted[msg.sender] = true;
+       quest.numberMinted++;
        emit ReceiptMinted(msg.sender, questId_);
        rabbitholeReceiptContract.mint(msg.sender, questId_);
    }
```

Also `hash_` argument can be removed, and be computed.

**Affected source code:**

- [QuestFactory.sol:219-229](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219-L229)

### Total gas saved:

In red the previous version, y green the new one:

```diff
mintReceipt
-     $43.22  (288,104) 1st mint
+     $43.02  (286,799) 1st mint
-     $41.49  (276,604) 2st mint
+     $41.29  (275,299) 2st mint
- Range: $41.49 - $43.22
+ Range: $41.29 - $43.02
- |  QuestFactory                                 ·          -  ·          -  ·     5313516  ·       [90m17.7 %[39m  ·          [32m[90m-[32m[39m  │
+ |  QuestFactory                                 ·          -  ·          -  ·     5286756  ·       [90m17.6 %[39m  ·          [32m[90m-[32m[39m  │
```

## **4. Optimize `getOwnedTokenIdsOfQuest`**

It is possible to optimize the gas cost of the `getOwnedTokenIdsOfQuest` method by avoiding the `keccak256` on each iteration.

```diff
    function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory) {
        uint msgSenderBalance = balanceOf(claimingAddress_);
+       if (msgSenderBalance == 0) return new uint[](0);
        uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
        uint foundTokens = 0;
+       bytes32 questHash = keccak256(bytes(questId_));
        for (uint i = 0; i < msgSenderBalance; i++) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
-           if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
+           if (keccak256(bytes(questIdForTokenId[tokenId])) == questHash) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
            }
        }

        uint[] memory filteredTokens = new uint[](foundTokens);
        uint filterTokensIndexTracker = 0;

        for (uint i = 0; i < msgSenderBalance; i++) {
            if (tokenIdsForQuest[i] > 0) {
                filteredTokens[filterTokensIndexTracker] = tokenIdsForQuest[i];
                filterTokensIndexTracker++;
            }
        }
        return filteredTokens;
    }
```

**Affected source code:**

- [RabbitHoleReceipt.sol:109-135](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L109-L135)

## **5. Reduce boolean comparison**

Compare a boolean value using `== true` or `== false` instead of using the boolean value is more expensive.

`NOT` opcode it's cheaper than using `EQUAL` or `NOTEQUAL` when the value it's false, or just the value without `== true` when it's true, because it will use less opcodes inside the VM.

**Proof of concept (*without optimizations*):**

```javascript
pragma solidity 0.8.16;

contract TesterA {
function testEqual(bool a) public view returns (bool) { return a == true; }
}

contract TesterB {
function testNot(bool a) public view returns (bool) { return a; }
}
```

Gas saving executing: **18 per entry for == true**

```
TesterA.testEqual:   21814
TesterB.testNot:     21796   
```

```javascript
pragma solidity 0.8.16;

contract TesterA {
function testEqual(bool a) public view returns (bool) { return a == false; }
}

contract TesterB {
function testNot(bool a) public view returns (bool) { return !a; }
}
```

Gas saving executing: **15 per entry for == false**

```
TesterA.testEqual:   21814
TesterB.testNot:     21799
```

**Affected source code:**

Use the value instead of `== true`:

- [Quest.sol:136](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L136)
- [QuestFactory.sol:73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73)
- [QuestFactory.sol:221](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221)

### Total gas saved:

In red the previous version, y green the new one:

```diff
claim
-     $19.00  (126,644) 1st claim
+     $18.99  (126,632) 1st claim
- Range: $19.00 - $19.00
+ Range: $18.99 - $18.99

mintReceipt
-     $43.22  (288,104) 1st mint
+     $43.21  (288,095) 1st mint
-     $41.49  (276,604) 2st mint
+     $41.49  (276,595) 2st mint
- Range: $41.49 - $43.22
+ Range: $41.49 - $43.21

···························
createQuest
-    $204.89 (1,365,913) Create ERC1155 quest
+    $204.59 (1,363,910) Create ERC1155 quest
-    $204.89 (1,365,913) Create ERC1155 quest
+    $204.59 (1,363,910) Create ERC1155 quest
-    $226.26 (1,508,398) Create ERC20 quest
+    $225.96 (1,506,382) Create ERC20 quest
-    $226.26 (1,508,398) Create ERC20 quest
+    $225.96 (1,506,382) Create ERC20 quest
- Range: $204.89 - $226.26
+ Range: $204.59 - $225.96
- |  QuestFactory                                 ·          -  ·          -  ·     5313516  ·       [90m17.7 %[39m  ·          [32m[90m-[32m[39m  │
+ |  QuestFactory                                 ·          -  ·          -  ·     5307411  ·       [90m17.7 %[39m  ·          [32m[90m-[32m[39m  │
``` 

## **6. Avoid compound assignment operator in state variables**

Using compound assignment operators for state variables (like `State += X` or `State -= X` ...) it's more expensive than using operator assignment (like `State = State + X` or `State = State - X` ...).

**Proof of concept (*without optimizations*):**

```javascript
pragma solidity 0.8.15;

contract TesterA {
uint256 private _a;
function testShort() public {
_a += 1;
}
}

contract TesterB {
Uint256 private _a;
function testLong() public {
_a = _a + 1;
}
}
```

Gas saving executing: **13 per entry**

```
TesterA.testShort: 43507
TesterB.testLong:  43494
```

**Affected source code:**

- [Quest.sol:115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115)

### Total gas saved:

In red the previous version, y green the new one:

```diff
 claim
-     $19.00  (126,644) 1st claim
+     $18.99  (126,628) 1st claim
-  Range: $19.00 - $19.00
+  Range: $18.99 - $18.99

  createQuest
-     $204.89 (1,365,913) Create ERC1155 quest
+     $204.71 (1,364,713) Create ERC1155 quest
-     $204.89 (1,365,913) Create ERC1155 quest
+     $204.71 (1,364,713) Create ERC1155 quest
-     $226.26 (1,508,398) Create ERC20 quest
+     $226.08 (1,507,194) Create ERC20 quest
-     $226.26 (1,508,398) Create ERC20 quest
+     $226.08 (1,507,194) Create ERC20 quest
- Range: $204.89 - $226.26
+ Range: $204.71 - $226.08
- |  QuestFactory                                 ·          -  ·          -  ·     5313516  ·       [90m17.7 %[39m  ·          [32m[90m-[32m[39m  │
+ |  QuestFactory                                 ·          -  ·          -  ·     5310936  ·       [90m17.7 %[39m  ·          [32m[90m-[32m[39m  │
```
