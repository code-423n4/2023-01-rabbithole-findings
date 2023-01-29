- [Low](#low)
    - [**1. Outdated compiler**](#1-outdated-compiler)
    - [**2. Lack of checks supportsInterface**](#2-lack-of-checks-supportsinterface)
- [Non critical](#non-critical)
    - [**3. Open TODO**](#3-open-todo)
    - [**4. Reentrancy pattern**](#4-reentrancy-pattern)

# Low

## **1. Outdated compiler**

The pragma version used are:

```
pragma solidity ^0.8.15;
```

*Note that mixing pragma is not recommended. Because different compiler versions have different meanings and behaviors, it also significantly raises maintenance costs. As a result, depending on the compiler version selected for any given file, deployed contracts may have security issues.*

The minimum required version must be [0.8.17](https://github.com/ethereum/solidity/releases/tag/v0.8.17); otherwise, contracts will be affected by the following **important bug fixes**:

[0.8.16](https://blog.soliditylang.org/2022/08/08/solidity-0.8.16-release-announcement/)

- Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

[0.8.17](https://blog.soliditylang.org/2022/09/08/solidity-0.8.17-release-announcement/)

- Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

Apart from these, there are several minor bug fixes and improvements.

## **2. Lack of checks `supportsInterface`**

The `EIP-165` standard helps detect that a smart contract implements the expected logic, prevents human error when configuring smart contract bindings, so it is recommended to check that the received argument is a contract and supports the expected interface.

**Reference:**

- https://eips.ethereum.org/EIPS/eip-165

**Affected source code:**

- [QuestFactory.sol:173](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L173)
- [Quest.sol:44](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L44)
- [Erc20Quest.sol:40](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L40)
- [RabbitHoleTickets.sol:44](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L44)
- [RabbitHoleTickets.sol:55](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L55)
- [RabbitHoleReceipt.sol:55](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L55)
- [RabbitHoleReceipt.sol:66](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L66)
- [RabbitHoleReceipt.sol:78](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L78)

---

# Non critical

## **3. Open TODO**

The code that contains "open todos" reflects that the development is not finished and that the code can change a posteriori, prior release, with or without audit.

**Affected source code:**

> // TODO clean this whole thing up

- [IQuest.sol:4](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L4)

## **4. Reentrancy pattern**

Although it has not been possible to exploit the reentrancy attack, the logic of the methods named below, make the changes of the validation flags after a method that allows reentrancy, it is convenient to modify the flags before the external calls to contracts.

In the `Quest.claim` method, the `redeemedTokens` state variable is modified after an external contract is called, so if that contract allows a reentrancy attack, the attack vector is possible. It's better to apply the following changes:

```diff
    ...
+       redeemedTokens += redeemableTokenCount;
        _transferRewards(totalRedeemableRewards);
-       redeemedTokens += redeemableTokenCount;
        emit Claimed(msg.sender, totalRedeemableRewards);
    }
```

**Affected source code:**

- [Quest.sol:115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L115)

