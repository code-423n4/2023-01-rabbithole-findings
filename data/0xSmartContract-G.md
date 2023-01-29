### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] | Remove the `initializer` modifier (80k  Gas) |4 |
| [G-02] |If the initialization() function is called by the wrong user, the implementation contract must be reloaded |3 |
| [G-03] |Deploy the contract with ``clone`` instead of ``new`` (x10 Gas Saved) |2 |
| [G-04] |Use inline assembly for ``recoverSigner`` function|1 |
| [G-05] |Gas optimization can be achieved by changing the control architecture in the ``createQuest`` function |2|
| [G-06] |Compute known value off-chain |2|
| [G-07] |Using ``delete``  instead of setting state variable ``0`` saves gas |1|
| [G-08] |Don’t compare boolean expressions to boolean literals |3|
| [G-09] |Change ``public`` function visibility to ``external`` |7|
| [G-10] |Use ``assembly`` to write _address storage values_  |12|
| [G-11] |Setting the _constructor_ to `payable` (13 gas)|6|
| [G-12] |Optimize names to save gas (22 gas) |All contracts|
| [G-13] |Empty blocks should be removed or emit something |3|
| [G-14] |Use a different pattern when using for loops|4|
| [G-15] |Save gas by naming returns in the ``getOwnedTokenIdsOfQuest`` function (8 gas) |1|
| [G-16] |`x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables |1|
| [G-17] |Use a more recent version of solidity |10|
| [G-18] |The solady Library's Ownable contract is significantly gas-optimized, which can be used |21|
| [G-19] |Use `Solmate SafeTransferLib ` contracts||
| [G-20] |Project file uses ``Transparent Proxy`` model but ``UUPS`` model is cheaper ||
| [G-21] |Use Minimalist and gas efficient standard ERC1155 implementation ||

Total 21 issues

### [G-01] Remove the `initializer` modifier (80k  Gas)

if we can just ensure that the `initialize()` function could only be called from within the constructor, we shouldn't need to worry about it getting called again. 

4 results - 3 files:
```solidity
contracts\QuestFactory.sol:
  35:     constructor() initializer {}

  41:     ) public initializer {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L35

```solidity
contracts\RabbitHoleReceipt.sol:
  48:     ) public initializer {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L48

```solidity
contracts\RabbitHoleTickets.sol:
  37:     ) public initializer {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L37

In the EVM, the constructor's job is actually to return the bytecode that will live at the contract's address. So, while inside a constructor, your address `(address(this))` will be the deployment address, but there will be no bytecode at that address! So if we check `address(this).code.length` before the constructor has finished, even from within a delegatecall, we will get 0. So now let's update our `initialize()` function to only run if we are inside a constructor:

```diff
contracts\QuestFactory.sol:
  36  
  37:     function initialize(
  38:         address claimSignerAddress_,
  39:         address rabbitholeReceiptContract_,
  40:         address protocolFeeRecipient_
  41:     ) public initializer {
+             require(address(this).code.length == 0, 'not in constructor');
```

Now the Proxy contract's constructor can still delegatecall initialize(), but if anyone attempts to call it again (after deployment) through the Proxy instance, or tries to call it directly on the above instance, it will revert because address(this).code.length will be nonzero. 

Also, because we no longer need to write to any state to track whether initialize() has been called, we can avoid the 20k storage gas cost. In fact, the cost for checking our own code size is only 2 gas, which means we have a 10,000x gas savings over the standard version. Pretty neat!

### [G-02] If the initialization() function is called by the wrong user, the implementation contract must be reloaded

If the `initialize()` function is run by another wrong user, its contract must be reloaded. This means gas cost.

3 results 3 files:
```solidity
contracts/QuestFactory.sol:
  37:     function initialize(
  38          address claimSignerAddress,
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L37

```solidity
contracts/RabbitHoleReceipt.sol:
  43:     function initialize(
  44          address receiptRenderer,
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L43

```solidity
contracts/RabbitHoleTickets.sol:
  32:     function initialize(
  33          address ticketRenderer_,
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L32


**Recommendation:**
Add a control that makes `initialize()` only call the Deployer Contract or EOA;

```solidity
if (msg.sender != DEPLOYER_ADDRESS) {
            revert NotDeployer();
        }
```

### [G-03] Deploy the contract with ``clone`` instead of ``new`` (x10 Gas Saved)

There’s a way to save a significant amount of gas on deployment using Clones: 
https://www.youtube.com/watch?v=3Mw-pMmJ7TA

It is extremely inexpensive to distribute by Cloneing with ``Create2``.

[Gas usage difference between the two? (answer: a new clone is 10x cheaper)](https://github.com/porter-finance/v1-core/issues/15#issuecomment-1035639516)


This is a solution that was adopted, as an example, by Porter Finance. They realized that deploying using clones was 10x cheaper:

Reference:  https://github.com/porter-finance/v1-core/pull/34

There are 2 examples of this issue:
```solidity
contracts\QuestFactory.sol:
  75:     Erc20Quest newQuest = new Erc20Quest(
  76:         rewardTokenAddress_,
  77:         endTime_,
  78:         startTime_,
  79:         totalParticipants_,
  80:         rewardAmountOrTokenId_,
  81:         questId_,
  82:         address(rabbitholeReceiptContract),
  83:         questFee,
  84:         protocolFeeRecipient
  85:     );
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L75-L85

```solidity
contracts\QuestFactory.sol:
  108:     Erc1155Quest newQuest = new Erc1155Quest(
  109:         rewardTokenAddress_,
  110:         endTime_,
  111:         startTime_,
  112:         totalParticipants_,
  113:         rewardAmountOrTokenId_,
  114:         questId_,
  115:         address(rabbitholeReceiptContract)
  116:     );
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L108-L116


### [G-04] Use inline assembly for ``recoverSigner`` function

``recoverSigner`` function is used to alot of time. So optimization of recoveSigner function is very efficent.

1 result - 1 file:
```solidity
contracts\QuestFactory.sol:
  209      /// @param signature_ The signature of the hash
  210:     function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) {
  211:         bytes32 messageDigest = keccak256(abi.encodePacked('\x19Ethereum Signed Message:\n32', hash_));
  212:         return ECDSAUpgradeable.recover(messageDigest, signature_);
  213:     }
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L210-L213

**Recommendation Code:**
```solidity
function recover(bytes32 hash, bytes calldata signature)
        internal
        view
        returns (address result)
    {
        /// @solidity memory-safe-assembly
        assembly {
            if eq(signature.length, 65) {
                // Copy the free memory pointer so that we can restore it later.
                let m := mload(0x40)
                // Directly copy `r` and `s` from the calldata.
                calldatacopy(0x40, signature.offset, 0x40)

                // If `s` in lower half order, such that the signature is not malleable.
                if iszero(gt(mload(0x60), _MALLEABILITY_THRESHOLD)) {
                    mstore(0x00, hash)
                    // Compute `v` and store it in the scratch space.
                    mstore(0x20, byte(0, calldataload(add(signature.offset, 0x40))))
                    pop(
                        staticcall(
                            gas(), // Amount of gas left for the transaction.
                            0x01, // Address of `ecrecover`.
                            0x00, // Start of input.
                            0x80, // Size of input.
                            0x40, // Start of output.
                            0x20 // Size of output.
                        )
                    )
                    // Restore the zero slot.
                    mstore(0x60, 0)
                    // `returndatasize()` will be `0x20` upon success, and `0x00` otherwise.
                    result := mload(sub(0x60, returndatasize()))
                }
                // Restore the free memory pointer.
                mstore(0x40, m)
            }
        }
    }
```


### [G-05] Gas optimization can be achieved by changing the control architecture in the ``createQuest`` function

It can be preferred to provide gas optimization by changing the control architecture in the ``createQuest`` function as seen in the code below.

2 results 1 files:
```diff
contracts\QuestFactory.sol:
  61:     function createQuest(
  62         address rewardTokenAddress_,
  63         uint256 endTime_,
  64         uint256 startTime_,
  65         uint256 totalParticipants_,
  66         uint256 rewardAmountOrTokenId_,
- 67         string memory contractType_,
+            bool isERC20,
  68         string memory questId_
  69     ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
  70         if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();
  71 
- 72:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
+             if (isERC20) {
  73             if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();


- 105:        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
+             if (!isERC20) {
  106:            if (msg.sender != owner()) revert OnlyOwnerCanCreate1155Quest();
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L105


### [G-06] Compute known value off-chain

If it is known which data to hash, no more computational power consumption is needed to hash using keccak256, it causes 2 times more gas consumption.

2 results - 1 file:
```solidity
contracts\QuestFactory.sol:
   72:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {

  105:         if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L105


**Recommendation Code:**
*contracts\QuestFactory.sol#L72*
```diff
         // keccak256(abi.encodePacked('erc20'))
+        bytes32 internal constant CONTRACT_TYPE_ERC20 = 0x5a28e9363bb942b639270062aa6bb295f434bcdfc42c97267bf003f272060dc9;

         //keccak256(abi.encodePacked('erc1155'))
+        bytes32 internal constant CONTRACT_TYPE_ERC1155 = 0xe511cd58b03579e6e411c64addfe164669e72b88eaa64705f148d9c3a2180fbc;

  61:    function createQuest(
- 72:       if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
+ 72:       if (keccak256(abi.encodePacked(contractType_)) == CONTRACT_TYPE_ERC20) {

- 105:      if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc1155'))) {
+ 105:      if (keccak256(abi.encodePacked(contractType_)) == CONTRACT_TYPE_ERC1155) {
```


### [G-07] Using ``delete``  instead of setting state variable ``0`` saves gas

1 result - 1 file:
```solidity
contracts\Quest.sol:
  26     constructor(
  45:        redeemedTokens = 0;
  46     }
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L45


**Recommendation code:**
```diff
  26     constructor(
- 45:        redeemedTokens = 0;
+ 45:        delete redeemedTokens;
  46     }
```

### [G-08] Don’t compare boolean expressions to boolean literals

```solidity
if (enabled) instead of if (enabled == true)
if (!enabled) instead of if (enabled == false)
```

Comparing to a constant (true or false) is a bit more expensive than directly checking the returned boolean value.

3 results - 2 files:
```solidity
contracts\Quest.sol:
  136:    return claimedList[tokenId_] == true;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L136

```solidity
contracts\QuestFactory.sol:
   73:    if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

  221:    if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73


**Recommendation:**
contracts\QuestFactory.sol#L73
```diff
-   73:    if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();
+   73:    if (!rewardAllowlist[rewardTokenAddress_]) revert RewardNotAllowed();
```

### [G-09] Change ``public`` function visibility to ``external``

Since the following functions are not called from other contracts in `RabbitHoleReceipt.sol` and `RabbitHoleTickets.sol` contracts, they can be made “external” in terms of gas optimization.

Reference: https://mudit.blog/solidity-gas-optimization-tips/

7 results - 2 files:
```solidity
contracts\RabbitHoleReceipt.sol:
  65:     function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
  66          ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);

  71:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
  72          royaltyRecipient = royaltyRecipient_;

  83:     function setMinterAddress(address minterAddress_) public onlyOwner {
  84          minterAddress = minterAddress_;

  90:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
  91          royaltyFee = royaltyFee_;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71-L72

```solidity
contracts\RabbitHoleTickets.sol:
  60:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
  61          royaltyRecipient = royaltyRecipient_;

  73:     function setMinterAddress(address minterAddress_) public onlyOwner {
  74          minterAddress = minterAddress_;

  66:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
  67          royaltyFee = royaltyFee_;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L60-L61


### [G-10] Use ``assembly`` to write _address storage values_ 

12 result - 3 files:
```solidity
contracts\QuestFactory.sol:
  37     function initialize(
  45:        claimSignerAddress = claimSignerAddress_;

  159    function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
  160:       claimSignerAddress = claimSignerAddress_;

  165    function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
  167:       protocolFeeRecipient = protocolFeeRecipient_;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L45

```solidity
contracts\RabbitHoleReceipt.sol:
  43     function initialize(
  52:        royaltyRecipient = royaltyRecipient_;
  53:        minterAddress = minterAddress_;

  71     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
  72:        royaltyRecipient = royaltyRecipient_;

  73     function setMinterAddress(address minterAddress_) public onlyOwner {
  74:        minterAddress = minterAddress_;

  83     function setMinterAddress(address minterAddress_) public onlyOwner {
  84:        minterAddress = minterAddress_;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L52-L53

```solidity
contracts\RabbitHoleTickets.sol:
  32     function initialize(
  41:        royaltyRecipient = royaltyRecipient_;
  42:        minterAddress = minterAddress_;

  60     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
  61:        royaltyRecipient = royaltyRecipient_;

  73     function setMinterAddress(address minterAddress_) public onlyOwner {
  74:        minterAddress = minterAddress_;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L41-L42



**Recommendation Code:**
contracts\RabbitHoleTickets.sol#L74
```diff
  73     function setMinterAddress(address minterAddress_) public onlyOwner {
- 74:        minterAddress = minterAddress_;
+                  assembly {                      
+                      sstore(minterAddress.slot, minterAddress_)
+                  }                               
```

### [G-11] Setting the _constructor_ to `payable` (13 gas)

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ```msg.value == 0``` and saves ```13 gas``` on deployment with no security risks.

6 results - 6 files:
```solidty
contracts\Erc20Quest.sol:
  17:     constructor(
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L17

```solidty
contracts\Erc1155Quest.sol:
  13:     constructor(
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L13

```solidty
contracts\Quest.sol:
  26:     constructor(
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L26

```solidty
contracts\QuestFactory.sol:
  35:     constructor() initializer {}
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L35

```solidty
contracts\RabbitHoleReceipt.sol:
  39:     constructor() {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L39

```solidty
contracts\RabbitHoleTickets.sol:
  28:     constructor() {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L28


**Recommendation:**
Set the constructor to ```payable```

### [G-12] Optimize names to save gas (22 gas)

Contracts most called functions could simply save gas by function ordering via ```Method ID```. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because ```22 gas``` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

**Context:** 
All Contracts


**Recommendation:** 
Find a lower ```method ID``` name for the most called functions for example Call() vs. Call1() is cheaper by ```22 gas```
For example, the function IDs in the ```Erc20Quest.sol``` contract will be the most used; A lower method ID may be given.

**Proof of Consept:**
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

Erc20Quest.sol function names can be named and sorted according to METHOD ID

```js
Sighash   |   Function Signature
========================
43865860  =>  _transferRewards(uint256)
4f51407c  =>  maxTotalRewards()
f4c17a6b  =>  maxProtocolReward()
be9a6555  =>  start()
bc0547e3  =>  _calculateRewards(uint256)
5affbd97  =>  withdrawRemainingTokens(address)
5215153b  =>  receiptRedeemers()
b0e21e8a  =>  protocolFee()
e941fa78  =>  withdrawFee()
```


### [G-13] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}). Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

3 results - 3 files:
```solidity
contracts\Erc1155Quest.sol:
  13     constructor(
  30:    ){}
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L30

```solidity
contracts\Quest.sol:
  150:    function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150

```solidity
contracts\QuestFactory.sol:
  35:     constructor() initializer {}
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L35


### [G-14] Use a different pattern when using for loops

In the use of for loops, this structure, which will reduce gas costs, can be preferred. It saves approximately 400 gas in an array with 6 members.

4 results - 2 files:
```solidity
contracts\Quest.sol:
   70:    for (uint i = 0; i < tokenIds_.length; i++) {

  104:    for (uint i = 0; i < tokens.length; i++) {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L70

```solidity
contracts\RabbitHoleReceipt.sol:
  117:    for (uint i = 0; i < msgSenderBalance; i++) {
 
  128:    for (uint i = 0; i < msgSenderBalance; i++) {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L117


**Recommendation code:**
```diff
contracts\RabbitHoleReceipt.sol#L117
- 117:    for (uint i = 0; i < msgSenderBalance; i++) {
+ 117:    for (uint i = 0; i < msgSenderBalance; i = unchecked_inc(i)) {

+ function unchecked_inc(uint i) internal pure returns (uint256) {
+        unchecked {
+            return i + 1;
+        }
+    }
```

### [G-15] Save gas by naming returns in the ``getOwnedTokenIdsOfQuest`` function (8 gas)

```diff
function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
-   ) public view returns (uint[] memory) {
+   ) public view returns (uint[] memory filteredTokens) {
       uint msgSenderBalance = balanceOf(claimingAddress_);     
        uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
        uint foundTokens = 0;

        for (uint i = 0; i < msgSenderBalance; i++) {
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
                tokenIdsForQuest[i] = tokenId;
                foundTokens++;
            }
        }

-       uint[] memory filteredTokens = new uint[](foundTokens);
+       filteredTokens = new uint[](foundTokens);
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

**Proof Of Concept:**

_Before Code:_
```js
·---------------------------------------------------|---------------------------|--------------|-----------------------------·
|               Solc version: 0.8.15                ·  Optimizer enabled: true  ·  Runs: 5000  ·  Block limit: 30000000 gas  │
····················································|···························|··············|······························
|  Methods                                                                                                                   │
······················|·····························|·············|·············|··············|···············|··············
|  Contract           ·  Method                     ·  Min        ·  Max        ·  Avg         ·  # calls      ·  eur (avg)  │
······················|·····························|·············|·············|··············|···············|··············
|  Erc1155Quest       ·  claim                      ·     126644  ·     155502  ·      131059  ·            7  ·          -  │
······················|·····························|·············|·············|··············|···············|··············
```


_After Code:_
```js
·---------------------------------------------------|---------------------------|--------------|-----------------------------·
|               Solc version: 0.8.15                ·  Optimizer enabled: true  ·  Runs: 5000  ·  Block limit: 30000000 gas  │
····················································|···························|··············|······························
|  Methods                                                                                                                   │
······················|·····························|·············|·············|··············|···············|··············
|  Contract           ·  Method                     ·  Min        ·  Max        ·  Avg         ·  # calls      ·  eur (avg)  │
······················|·····························|·············|·············|··············|···············|··············
|  Erc1155Quest       ·  claim                      ·     126636  ·     155494  ·      131051  ·            7  ·          -  │
······················|·····························|·············|·············|··············|···············|··············
```

### [G-16] `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables

1 result - 1 file:
```diff
contracts\Quest.sol:

- 115:    redeemedTokens += redeemableTokenCount;
+ 115:    redeemedTokens = redeemedTokens  + redeemableTokenCount;
```

`x += y (x -= y)` costs more gas than `x = x  + y (x = x - y)` for state variables.


### [G-17] Use a more recent version of solidity

> Solidity 0.8.10 has a useful change that reduced gas costs of external calls which expect a return value. 
> 
> In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.
> 
> In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

The version of 10 contracts included in the scope is ``0.8.15``. I recommend that you upgrade the versions of all contracts in scope to the latest version of robustness, '0.8.17’.

10 results - 10 files:
```solidity
contracts\Erc20Quest.sol:
  2: pragma solidity ^0.8.15;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L2

```solidity
contracts\Erc1155Quest.sol:
  2: pragma solidity ^0.8.15;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L2

```solidity
contracts\Quest.sol:
  2: pragma solidity ^0.8.15;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L2

```solidity
contracts\QuestFactory.sol:
  2: pragma solidity ^0.8.15; 
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L2

```solidity
contracts\RabbitHoleReceipt.sol:
  2: pragma solidity ^0.8.15;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L2

```solidity
contracts\RabbitHoleTickets.sol:
  2: pragma solidity ^0.8.15;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L2

```solidity
contracts\ReceiptRenderer.sol:
  2: pragma solidity ^0.8.15; 
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L2

```solidity
contracts\TicketRenderer.sol:
  2: pragma solidity ^0.8.15;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L2

```solidity
contracts\interfaces\IQuest.sol:
  2: pragma solidity ^0.8.15;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L2

```solidity
contracts\interfaces\IQuestFactory.sol:
  2: pragma solidity ^0.8.15;
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuestFactory.sol#L2

### [G-18] The solady Library's Ownable contract is significantly gas-optimized, which can be used

The project uses the ` onlyOwner ` authorization model I recommend using _Solady's highly gas optimized contract._

https://github.com/Vectorized/solady/blob/main/src/auth/OwnableRoles.sol

21 results - 7 files:
```solidity
contracts\Erc20Quest.sol:
  81:     function withdrawRemainingTokens(address to_) public override onlyOwner {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81

```solidity
contracts\Erc1155Quest.sol:
  54:     function withdrawRemainingTokens(address to_) public override onlyOwner {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54

```solidity
contracts\Quest.sol:
   50:     function start() public virtual onlyOwner {
   57:     function pause() public onlyOwner onlyStarted {
   63:     function unPause() public onlyOwner onlyStarted {
  150:     function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50

```solidity
contracts\QuestFactory.sol:
  142:     function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
  159:     function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
  165:     function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
  172:     function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
  179:     function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
  186:     function setQuestFee(uint256 questFee_) public onlyOwner {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L142

```solidity
contracts\RabbitHoleReceipt.sol:
  65:     function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
  71:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
  77:     function setQuestFactory(address questFactory_) public onlyOwner {
  83:     function setMinterAddress(address minterAddress_) public onlyOwner {
  90:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L65

```solidity
contracts\RabbitHoleTickets.sol:
  54:     function setTicketRenderer(address ticketRenderer_) public onlyOwner {
  60:     function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
  66:     function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
  73:     function setMinterAddress(address minterAddress_) public onlyOwner {```
```
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L54


### [G-19] Use `Solmate SafeTransferLib ` contracts

**Description:**
Use the gas-optimized Solmate SafeTransferLib contract for Erc20

https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol


### [G-20] Project file uses ``Transparent Proxy`` model but ``UUPS`` model is cheaper


UUPS proxies are a lot simpler than Transparent, and they only have the logic for delegating the call. This makes them much cheaper to deploy, and each all has half the overhead (just a single SLOAD).

Reference: https://twitter.com/smpalladino/status/1389939166109212675?s=20&t=Nd7ssD9sC_BNTtQF8lnxNg

### [G-21] Use Minimalist and gas efficient standard ERC1155 implementation


I recommend using the minimalist and gas-efficient standard ERC1155 implementation.

Reference: https://github.com/transmissions11/solmate/blob/main/src/tokens/ERC1155.sol


