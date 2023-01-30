## 1. `quests[questId_]` should be cached into a variable when read multiple times to save gas

For example:

File: `QuestFactory.sol` [Line 219-229](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219-L229)

```solidity
    function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
        if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
        if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
        if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();

        quests[questId_].addressMinted[msg.sender] = true;
        quests[questId_].numberMinted++;
        emit ReceiptMinted(msg.sender, questId_);
        rabbitholeReceiptContract.mint(msg.sender, questId_);
    }
```

In the above, instead of reading `quests[questId_]` multiple times, it could be cached in a variable to save gas.

Here is how it would be implemented:

```solidity
    function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
        Quest storage quest = quests[questId_];

        if (quest.numberMinted + 1 > quest.totalParticipants) revert OverMaxAllowedToMint();
        if (quest.addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
        if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
        if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();

        quest.addressMinted[msg.sender] = true;
        quest.numberMinted++;
        emit ReceiptMinted(msg.sender, questId_);
        rabbitholeReceiptContract.mint(msg.sender, questId_);
    }
```

On average, just that change will save **1305 gas** per call:

```diff
·----------------------------------------------|---------------------------|--------------|-----------------------------·
|             Solc version: 0.8.15             ·  Optimizer enabled: true  ·  Runs: 5000  ·  Block limit: 30000000 gas  │
···············································|···························|··············|······························
|  Methods                                                                                                              │
·················|·····························|·············|·············|··············|···············|··············
|  Contract      ·  Method                     ·  Min        ·  Max        ·  Avg         ·  # calls      ·  eur (avg)  │
·················|·····························|·············|·············|··············|···············|··············
|  QuestFactory  ·  changeCreateQuestRole      ·          -  ·          -  ·       56104  ·            1  ·          -  │
·················|·····························|·············|·············|··············|···············|··············
|  QuestFactory  ·  createQuest                ·    1365853  ·    1508346  ·     1472723  ·            8  ·          -  │
·················|·····························|·············|·············|··············|···············|··············
- |  QuestFactory  ·  mintReceipt                ·          -  ·          -  ·      288053  ·            2  ·          -  │
+ |  QuestFactory  ·  mintReceipt                ·          -  ·          -  ·      286748  ·            2  ·          -  │
·················|·····························|·············|·············|··············|···············|··············
|  QuestFactory  ·  setClaimSignerAddress      ·          -  ·          -  ·       36379  ·            1  ·          -  │
·················|·····························|·············|·············|··············|···············|··············
|  QuestFactory  ·  setProtocolFeeRecipient    ·          -  ·          -  ·       36347  ·            1  ·          -  │
·················|·····························|·············|·············|··············|···············|··············
|  QuestFactory  ·  setQuestFee                ·          -  ·          -  ·       36037  ·            1  ·          -  │
·················|·····························|·············|·············|··············|···············|··············
|  QuestFactory  ·  setRewardAllowlistAddress  ·      31826  ·      53738  ·       50608  ·            7  ·          -  │
·················|·····························|·············|·············|··············|···············|··············
|  Deployments                                 ·                                          ·  % of limit   ·             │
···············································|·············|·············|··············|···············|··············
|  QuestFactory                                ·          -  ·          -  ·     5313516  ·       17.7 %  ·          -  │
···············································|·············|·············|··············|···············|··············
|  RabbitHoleReceipt                           ·          -  ·          -  ·     2776002  ·        9.3 %  ·          -  │
···············································|·············|·············|··············|···············|··············
|  ReceiptRenderer                             ·          -  ·          -  ·     1186932  ·          4 %  ·          -  │
···············································|·············|·············|··············|···············|··············
|  SampleErc1155                               ·          -  ·          -  ·     1417938  ·        4.7 %  ·          -  │
···············································|·············|·············|··············|···············|··············
|  SampleERC20                                 ·          -  ·          -  ·      737168  ·        2.5 %  ·          -  │
·----------------------------------------------|-------------|-------------|--------------|---------------|-------------·
```

There are 2 other instances of this issue:

File: [`QuestFactory.sol`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61)

```solidity
File: contracts/QuestFactory.sol

       /// @audit quests[questId_] can be cached in a variable
61:    function createQuest(
62:        address rewardTokenAddress_,
63:        uint256 endTime_,
64:        uint256 startTime_,
65:        uint256 totalParticipants_,
66:        uint256 rewardAmountOrTokenId_,
67:        string memory contractType_,
68:        string memory questId_
69:    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
70:        if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();
...
98:            quests[questId_].questAddress = address(newQuest);
99:            quests[questId_].totalParticipants = totalParticipants_;
...
129:            quests[questId_].questAddress = address(newQuest);
130:            quests[questId_].totalParticipants = totalParticipants_;
...
137:    }

        /// @audit quests[questId_] can be cached in a variable
199:    function questInfo(string memory questId_) external view returns (address, uint, uint) {
200:        return (
201:            quests[questId_].questAddress,
202:            quests[questId_].totalParticipants,
203:            quests[questId_].numberMinted
204:        );
205:    }
```

## 2. Comparison with boolean literal values

You will save gas by not comparing boolean literals (i.e. `true` and `false`)

There are 3 instances of this issue:

File: `QuestFactory.sol` ([Line 73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L73), [Line 221](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L221))

```solidity
73:            if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();

221:        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
```

File: `Quest.sol` [Line 136](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L136)

```solidity
136:        return claimedList[tokenId_] == true;
```

## 3. On/off functions can be merged into a single toggle function to save gas

For example:

File: `Quest.sol` [Line 55-65](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L55-L65)

```solidity
55:    /// @notice Pauses the Quest
56:    /// @dev Only the owner of the Quest can call this function. Also requires that the Quest has started (not by date, but by calling the start function)
57:    function pause() public onlyOwner onlyStarted {
58:        isPaused = true;
59:    }
60:
61:    /// @notice Unpauses the Quest
62:    /// @dev Only the owner of the Quest can call this function. Also requires that the Quest has started (not by date, but by calling the start function)
63:    function unPause() public onlyOwner onlyStarted {
64:        isPaused = false;
65:    }
```

Instead, of `pause()` and `unpause()` functions, the two can be merged into a `setPaused()` function:

```solidity
    /// @notice Sets the Pause status for the Quest
    /// @dev Only the owner of the Quest can call this function. Also requires that the Quest has started
    function setPaused(bool isPaused_) public onlyOwner onlyStarted {
        isPaused = isPaused_;
    }
```

## 4. Avoid unnecessarily setting variables to save gas

`redeemedTokens` is unnecessarily set to 0 in the `constructor()` when its value is already 0:

File: [`Quest.sol`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L22)

```solidity
File: contracts/Quest.sol

22:    uint256 public redeemedTokens;
...
26:    constructor(
            ...
34:    ) {
...
45:        redeemedTokens = 0;
46:    }
```

Because `redeemedTokens` is already set to an initial value of 0, setting it to 0 again is unnecessary. As such, [line 45](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L45) can be removed to save gas.

## 5. Using `storage` instead of `memory` for structs/arrays saves gas

When retrieving data from a `storage` location, assigning the data to a `memory` variable will cause all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declaring the variable with the memory keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incurring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct

For example, the following `memory` variables could be changed to `storage`:

File: `RabbitHoleReceipt.sol` ([Line 114](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L114), [Line 125](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L125))

```solidity
114:        uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);

125:        uint[] memory filteredTokens = new uint[](foundTokens);
```

## 6. Unnecessary `onlyOwner` modifier

In [`Erc1155Quest.sol`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54) and [`Erc20Quest.sol`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81), the function `withdrawRemainingTokens()` invokes `super.withdrawRemainingTokens()`. Since [the parent function](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L150) already has the `onlyOwner` modifier, putting it on the child contract is unnecessary and will waste gas.

Since `super.withdrawRemainingTokens()` already calls the `onlyOwner` modifier, consider removing the `onlyOwner` modifier on the following functions:

- File: `Erc20Quest.sol` [Line 81 - Line 87](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81-L87)
- File: `Erc1155Quest.sol` [Line 54 - Line 63](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54-L63)
