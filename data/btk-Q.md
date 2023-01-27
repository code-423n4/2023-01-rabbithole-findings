| Total Low issues   |
|--------------------|

| Risk   | Issues Details                                                | Number        |
|--------|---------------------------------------------------------------|---------------|
| [L-01] | `Address(0)` checks                                           | 6             |
| [L-02] | Missing Event for critical changes                            | 15            |
| [L-03] | Loss of precision due to rounding                             | 4             |
| [L-04] | `onlyMinter()` modifier is not working as expected            | 2             |
| [L-05] | Critical changes should use-two step procedure                | 2             |
| [L-06] | No storage gap for upgradeable contracts                      | 3             |
| [L-07] | `royaltyFee` is not capped at 100%                            | 2             |
| [L-08] | Protocol is using deprecated `npm` dependency (axios)         | 1             |
| [L-09] | Value is not validated to be different than the existing one  | 4             |
| [L-10] | Add a timelock to critical functions                          | 3             |


| Total Non-Critical issues   |
|-----------------------------|

| Risk    | Issues Details                                                                            | Number        |
|---------|-------------------------------------------------------------------------------------------|---------------|
| [NC-01] | Use `uint256` instead `uint`                                                              | 33            |
| [NC-02] | Lock pragmas to specific compiler version                                                 | All Contracts |
| [NC-03] | Use a more recent version of solidity                                                     | All Contracts |
| [NC-04] | Constants in comparisons should appear on the left side                                   | 2             |
| [NC-05] | Solidity compiler optimizations can be problematic                                        | 1             |
| [NC-06] | Use `immutable` instead of `constant` for values such as a call to `keccak256()`          | 1             |
| [NC-07] | Events that mark critical parameter changes should contain both the old and the new value | 4             |
| [NC-08] | Use `bytes.concat()` and `string.concat()`                                                | 16            |
| [NC-09] | Non-usage of specific imports                                                             | All Contracts |
| [NC-10] | Mark visibility of `initialize()` functions as external                                   | 3             |
| [NC-11] | Include `@return` parameters in NatSpec comments                                          | All Contracts |
| [NC-12] | Function writing does not comply with the `Solidity Style Guide`                          | All Contracts |
| [NC-13] | Lines are too long                                                                        | 6             |
| [NC-14] | NatSpec comments should be increased in contracts                                         | All Contracts |
| [NC-15] | For functions, follow Solidity standard naming conventions                                | All Contracts |
| [NC-16] | Contracts should have full test coverage                                                  | All Contracts |
| [NC-17] | Critical changes should use-two step procedure                                            | 3             |
| [NC-18] | Generate perfect code headers every time                                                  | All Contracts |
| [NC-19] | There is no need to cast a variable that is already an address, such as `address(x)`      | 2             |
| [NC-20] | Add NatSpec comment to `mapping`                                                          | 6             |


## [L-01] `Address(0)` checks

#### Description

Check of `address(0)` to protect the code from `(0x0000000000000000000000000000000000000000)` address problem just in case. This is best practice or instead of suggesting that they verify `_address != address(0)`, you could add some good NatSpec comments explaining what is valid and what is invalid and what are the implications of accidentally using an invalid address.

```solidity
    function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
        claimSignerAddress = claimSignerAddress_;
    }
```

#### Lines of code 

- [QuestFactory.sol:172](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L172)
- [RabbitHoleReceipt.sol:65](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L65)
- [RabbitHoleReceipt.sol:77](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L77)
- [RabbitHoleReceipt.sol:83](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L83)
- [RabbitHoleTickets.sol:54](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L54)
- [RabbitHoleTickets.sol:73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L73)

#### Recommended Mitigation Steps

```solidity
    function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
        if (claimSignerAddress_ == address(0)) revert();
        claimSignerAddress = claimSignerAddress_;
    }
```

## [L-02] Missing Event for critical changes

#### Description

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

```solidity
    function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
        if (protocolFeeRecipient_ == address(0)) revert AddressZeroNotAllowed();
        protocolFeeRecipient = protocolFeeRecipient_;
    }
```

#### Lines of code 

- [QuestFactory.sol:159](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159)
- [QuestFactory.sol:165](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L165)
- [QuestFactory.sol:172](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L172)
- [QuestFactory.sol:179](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L179)
- [QuestFactory.sol:186](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L186)
- [RabbitHoleReceipt.sol:65](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L65)
- [RabbitHoleReceipt.sol:71](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71)
- [RabbitHoleReceipt.sol:77](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L77)
- [Quest.sol:50](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50)
- [Quest.sol:57](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57)
- [Quest.sol:63](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L63)
- [RabbitHoleTickets.sol:54](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L54)
- [RabbitHoleTickets.sol:60](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L60)
- [Erc20Quest.sol:102](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L102)
- [Erc1155Quest.sol:54](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54)

#### Recommended Mitigation Steps

Emit event for critical changes.

## [L-03] Loss of precision due to rounding

#### Description

Loss of precision due to the nature of arithmetics and rounding errors.

```solidity
        uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
```

#### Lines of code 

- [RabbitHoleReceipt.sol:184](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L184)
- [RabbitHoleTickets.sol:113](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L113)
- [Erc20Quest.sol:53](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L53)
- [Erc20Quest.sol:97](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L97)

#### Recommended Mitigation Steps

Add scalars so roundings are negligible.

## [L-04] `onlyMinter()` modifier is not working as expected

#### Description

`onlyMinter()` can be bypasssed by anyone due to an invalid check: 

```solidity
    modifier onlyMinter() {
        msg.sender == minterAddress;
        _;
    }
```

Thus, everyone can mint tokens: 

```solidity
    function mint(address to_, string memory questId_) public onlyMinter {
        _tokenIds.increment();
        uint newTokenID = _tokenIds.current();
        questIdForTokenId[newTokenID] = questId_;
        timestampForTokenId[newTokenID] = block.timestamp;
        _safeMint(to_, newTokenID);
    }
```

#### Lines of code 

- [RabbitHoleReceipt.sol:58](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L58)
- [RabbitHoleTickets.sol:47](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L47)

#### Recommended Mitigation Steps

Replace the affected modifier by this one: 

```solidity
    modifier onlyMinter() {
        require(msg.sender == minterAddress, "Only minter can mint tokens");
        _;
    }
```

## [L-05] Critical changes should use two-step procedure   

#### Description

The protocol inherit openzeppelin [`OwnableUpgradeable.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/OwnableUpgradeable.sol) which does not use two-step procedure when transferring ownership, and since the protocol rely heavily on the `onlyOwner()` modifier (24 results on 8 files), Thus using two-step procedure is a best practice
in case of transferring the ownership to an invalid address.

```solidity
    function transferOwnership(address newOwner) public virtual onlyOwner {
        require(newOwner != address(0), "Ownable: new owner is the zero address");
        _transferOwnership(newOwner);
    }

    function _transferOwnership(address newOwner) internal virtual {
        address oldOwner = _owner;
        _owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }
```

#### Lines of code 

- [RabbitHoleReceipt.sol:7](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L7)
- [RabbitHoleTickets.sol:5](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L5)

#### Recommended Mitigation Steps

Consider adding two step procedure on the critical functions where the first is announcing a pending owner and the new address should then claim its ownership, or use [`Ownable2StepUpgradeable.sol`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol#L20) instead.

## [L-06] No storage gap for upgradeable contracts

#### Description

For upgradeable contracts, inheriting contracts may introduce new variables. In order to be able to add new variables to the upgradeable contract without causing storage collisions, a storage gap should be added to the upgradeable contract.

In order to be able to add new variables to the upgradeable contract without causing storage collisions, a storage gap should be added to the upgradeable contract.

See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) for a description of this storage variable. 

#### Lines of code 

- [QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)
- [RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol)
- [RabbitHoleTickets.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol)

#### Recommended Mitigation Steps

Consider adding a storage gap at the end of the upgradeable contract:

```solidity
  /**
   * @dev This empty reserved space is put in place to allow future versions to add new
   * variables without shifting down storage in the inheritance chain.
   * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
   */
  uint256[50] private __gap;
```

## [L-07] `royaltyFee` is not capped at 100%

#### Description

The `royaltyFee` is not capped at 100%, Thus it may exceed the `salePrice` which is not fully compatible with the `ERC-2981` resulting in:

- Giving the protocol a bad reputation.
- Users lost of funds.

```solidity
    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
```

#### Lines of code 

- [RabbitHoleTickets.sol:66](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66)
- [RabbitHoleReceipt.sol:90](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90)

#### Recommended Mitigation Steps

Add a timelock to the `setRoyaltyFee()` function, or add a `feeDenominator()` function:

```solidity
    /**
     * @dev The denominator with which to interpret the fee set in {_setTokenRoyalty} and {_setDefaultRoyalty} as a
     * fraction of the sale price. Defaults to 10000 so fees are expressed in basis points, but may be customized by an
     * override.
     */
    function _feeDenominator() internal pure virtual returns (uint96) {
        return 10000;
    }
```

and cap the `royaltyFee` to 100% :

```solidity
    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        require(royaltyFee_ <= _feeDenominator(), "ERC2981: royalty fee will exceed salePrice")
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
```

## [L-08] Protocol is using deprecated `npm` dependency (axios)

#### Description

The protocol is using deprecated `npm` dependency `(axios : 0.20.0)`

> Ref : https://www.npmjs.com/package/axios/v/0.20.0-0

```solidity
    "axios": "^0.20.0",
```

#### Lines of code 

- [package.json:49](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/package.json#L49)

#### Recommended Mitigation Steps

Upgrade `axios` to version `0.21.3` or higher.

## [L-09] Value is not validated to be different than the existing one

#### Description

While the value is validated to be in the constant bounds, it is not validated to be different than the existing one. Queueing the same value will cause multiple abnormal events to be emitted, will ultimately result in a no-op situation.

```solidity
    function setMinterAddress(address minterAddress_) public onlyOwner {
        minterAddress = minterAddress_;
        emit MinterAddressSet(minterAddress_);
    }
```

#### Lines of code 

- [RabbitHoleReceipt.sol:83](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L83)
- [RabbitHoleReceipt.sol:90](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90)
- [RabbitHoleTickets.sol:66](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66)
- [RabbitHoleTickets.sol:73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L73)

#### Recommended Mitigation Steps

```solidity
    function setMinterAddress(address minterAddress_) public onlyOwner {
        require(minterAddress_ != minterAddress, "Address the same as before")
        minterAddress = minterAddress_;
        emit MinterAddressSet(minterAddress_);
    }
```

## [L-10] Add a timelock to critical functions

#### Description

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate.

```solidity
    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
```

#### Lines of code 

- [QuestFactory.sol:186](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L186)
- [RabbitHoleReceipt.sol:90](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90)
- [RabbitHoleTickets.sol:66](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66)

#### Recommended Mitigation Steps

Consider adding a timelock to the critical changes.

## [NC-01] Use `uint256` instead `uint`

#### Description

`33 results in 7 files`.

Some developers prefer to use `uint256` because it is consistent with other `uint` data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs.

For example if doing `bytes4(keccak('transfer(address, uint)’))`, you'll get a different method sig ID than `bytes4(keccak('transfer(address, uint256)’))` and smart contracts will only understand the latter when comparing method sig IDs.

#### Lines of code 

- [Erc20Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol)
- [Quest.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol)
- [RabbitHoleReceipt.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol)
- [QuestFactory.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol)
- [RabbitHoleTickets.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol)
- [ReceiptRenderer.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol)
- [TicketRenderer.sol](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol)

#### Recommended Mitigation Steps

Use `uint256` instead `uint`.

## [NC-02] Lock pragmas to specific compiler version

#### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

> Ref: https://swcregistry.io/docs/SWC-103

```solidity
    pragma solidity ^0.8.15;
```

#### Lines of code 

- [All Contracts](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts)

#### Recommended Mitigation Steps

[Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/):  Lock pragmas to specific compiler version.

## [NC-03] Use a more recent version of solidity

#### Description

For security, it is best practice to use the [latest Solidity version](https://github.com/ethereum/solidity/blob/develop/Changelog.md).

```solidity
    pragma solidity ^0.8.15;
```

#### Lines of code 

- [All Contracts](https://github.com/rabbitholegg/quest-protocol/tree/8c4c1f71221570b14a0479c216583342bd652d8d/contracts)

#### Recommended Mitigation Steps

Old version of Solidity is used `(0.8.15)`, newer version can be used `(0.8.17)`.

## [NC-04] Constants in comparisons should appear on the left side

#### Description

Constants in comparisons should appear on the left side, doing so will prevent typo [bug](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html).

```solidity
        if (tokens.length == 0) revert NoTokensToClaim();
```

```solidity
        if (redeemableTokenCount == 0) revert AlreadyClaimed();
```

#### Lines of code 

- [Quest.sol:101](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L101)
- [Quest.sol:110](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L110)

#### Recommended Mitigation Steps

```solidity
        if (0 == tokens.length) revert NoTokensToClaim();
```

```solidity
        if (0 == redeemableTokenCount) revert AlreadyClaimed();
```

## [NC-05] Solidity compiler optimizations can be problematic

#### Description

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

```solidity
  solidity: {
    compilers: [
      {
        version: '0.8.15',
        settings: {
          optimizer: {
            enabled: true,
            runs: 5000,
          },
        },
```

#### Lines of code 

- [hardhat.config.ts:30](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/hardhat.config.ts#L30)

#### Recommended Mitigation Steps

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## [NC-06] Use `immutable` instead of `constant` for values such as a call to `keccak256()`	

#### Description

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand. There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts. constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

```solidity
    bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
```

#### Lines of code 

- [QuestFactory.sol:17](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L17)

#### Recommended Mitigation Steps

Use `immutable` instead of `constant`.

## [NC-07] Events that mark critical parameter changes should contain both the old and the new value

#### Description

Events that mark critical parameter changes should contain both the old and the new value.

```solidity
    function setMinterAddress(address minterAddress_) public onlyOwner {
        minterAddress = minterAddress_;
        emit MinterAddressSet(minterAddress_);
    }
```

#### Lines of code 

- [RabbitHoleReceipt.sol:83](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L83)
- [RabbitHoleReceipt.sol:90](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L90)
- [RabbitHoleTickets.sol:66](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L66)
- [RabbitHoleTickets.sol:73](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L73)

#### Recommended Mitigation Steps

```solidity
    
    event MinterAddressSet(address indexed previousMinterAddress, address indexed minterAddress);

    function setMinterAddress(address minterAddress_) public onlyOwner {
        emit MinterAddressSet(minterAddress, minterAddress_);
        minterAddress = minterAddress_;
    }
```

## [NC-08] Use `bytes.concat()` and `string.concat()`

#### Description

- `bytes.concat()` vs `abi.encodePacked(<bytes>,<bytes>)`
- `string.concat()` vs `abi.encodePacked(<string>,<string>)`

> https://docs.soliditylang.org/en/v0.8.17/types.html?highlight=bytes.concat#the-functions-bytes-concat-and-string-concat

#### Lines of code 

- [QuestFactory.sol:72](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L72)
- [QuestFactory.sol:105](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L105)
- [QuestFactory.sol:211](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L211)
- [QuestFactory.sol:222](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L222)
- [ReceiptRenderer.sol:37](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/ReceiptRenderer.sol#L37)
- [ReceiptRenderer.sol:48](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/ReceiptRenderer.sol#L48)
- [ReceiptRenderer.sol:63](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/ReceiptRenderer.sol#L63)
- [ReceiptRenderer.sol:83](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/ReceiptRenderer.sol#L83)
- [ReceiptRenderer.sol:101](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/ReceiptRenderer.sol#L101)
- [ReceiptRenderer.sol:113](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/ReceiptRenderer.sol#L113)
- [TicketRenderer.sol:19](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/TicketRenderer.sol#L19)
- [TicketRenderer.sol:30](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/TicketRenderer.sol#L30)
- [TicketRenderer.sol:37](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/TicketRenderer.sol#L37)
- [TicketRenderer.sol:46](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/TicketRenderer.sol#L46)

#### Recommended Mitigation Steps

Use `bytes.concat()` and `string.concat()` instead.

## [NC-09] Non-usage of specific imports

#### Description

Using import declarations of the form import `{<identifier_name>} from "some/file.sol"` avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation.

The Solidity docs recommend specifying imported symbols explicitly.

> Ref: https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

#### Lines of code 

- [All Contracts](https://github.com/rabbitholegg/quest-protocol/tree/main/contracts)

#### Recommended Mitigation Steps

Use specific imports syntax per solidity [docs](https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files) recommendation.

## [NC-10] Mark visibility of `initialize()` functions as external

#### Description

- If someone wants to extend via inheritance, it might make more sense that the overridden `initialize()` function calls the internal {...}_init function, not the parent public `initialize()` function.

- External instead of public would give more the sense of the `initialize()` functions to behave like a constructor (only called on deployment, so should only be called externally)

- Security point of view, it might be safer so that it cannot be called internally by accident in the child contract

- It might cost a bit less gas to use external over public

- It is possible to override a function from external to public ("opening it up") but it is not possible to override a function from public to external ("narrow it down").

> Ref: OpenZeppelin/openzeppelin-contracts#3750

```solidity
    function initialize(
        address ticketRenderer_,
        address royaltyRecipient_,
        address minterAddress_,
        uint royaltyFee_
    ) public initializer {
        __ERC1155_init('');
        __Ownable_init();
        __ERC1155Burnable_init();
        royaltyRecipient = royaltyRecipient_;
        minterAddress = minterAddress_;
        royaltyFee = royaltyFee_;
        TicketRendererContract = TicketRenderer(ticketRenderer_);
    }
```

#### Lines of code 

- [QuestFactory.sol:41](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/QuestFactory.sol#L41)
- [RabbitHoleReceipt.sol:48](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleReceipt.sol#L48)
- [RabbitHoleTickets.sol:37](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L37)

#### Recommended Mitigation Steps

Change the visibility of `initialize()` functions to external

## [NC-11] 	Include `@return` parameters in NatSpec comments

#### Description

If Return parameters are declared, you must prefix them with `@return`. Some code analysis programs do analysis by reading [NatSpec](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html) details, if they can't see the `@return` tag, they do incomplete analysis.

#### Lines of code 

- [All Contracts](https://github.com/rabbitholegg/quest-protocol/tree/main/contracts)

#### Recommended Mitigation Steps

Include the `@return` argument in the NatSpec comments.

## [NC-12] Function writing does not comply with the `Solidity Style Guide`

#### Description

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

Functions should be grouped according to their visibility and ordered:

- `constructor()`
- `receive()`  
- `fallback()`  
- `external` / `public` / `internal` / `private`
- `view` / `pure`

#### Lines of code 

- [All Contracts](https://github.com/rabbitholegg/quest-protocol/tree/main/contracts)

#### Recommended Mitigation Steps

Follow [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html?highlight=Style#style-guide).

## [NC-13] Lines are too long

#### Description

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

> Ref: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

#### Lines of code 

- [Erc20Quest.sol:56](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Erc20Quest.sol#L56)
- [Erc20Quest.sol:57](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Erc20Quest.sol#L57)
- [Erc20Quest.sol:79](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Erc20Quest.sol#L79)
- [Quest.sol:56](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Quest.sol#L56)
- [Quest.sol:62](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/Quest.sol#L62)
- [IQuestFactory.sol:16](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/interfaces/IQuestFactory.sol#L16)

#### Recommended Mitigation Steps

Split the long lines when they reach the max length.

## [NC-14] NatSpec comments should be increased in contracts

#### Description

It is recommended that Solidity contracts are fully annotated using NatSpec, it is clearly stated in the Solidity official documentation.

- In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

- Some code analysis programs do analysis by reading NatSpec details, if they can't see the tags `(@param, @dev, @return)`, they do incomplete analysis.

#### Lines of code 

- [All Contracts](https://github.com/rabbitholegg/quest-protocol/tree/main/contracts)

#### Recommended Mitigation Steps

Include [NatSpec](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html#natspec-format) comments in the codebase.

## [NC-15] For functions, follow Solidity standard naming conventions

#### Description

The protocol don't follow solidity standard naming convention.

> Ref: https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions

#### Lines of code 

- [All Contracts](https://github.com/rabbitholegg/quest-protocol/tree/main/contracts)

#### Recommended Mitigation Steps

Follow solidity standard naming convention.

## [NC-16] Contracts should have full test coverage

#### Description

While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

```solidity
- What is the overall line coverage percentage provided by your tests?: 89 
```

#### Lines of code 

- [All Contracts](https://github.com/rabbitholegg/quest-protocol/tree/main/contracts)

#### Recommended Mitigation Steps

Line coverage percentage should be 100%.

## [NC-17] Critical changes should use-two step procedure

#### Description

Critical changes should use-two step procedure.

```solidity
    function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
        if (protocolFeeRecipient_ == address(0)) revert AddressZeroNotAllowed();
        protocolFeeRecipient = protocolFeeRecipient_;
    }
```

#### Lines of code 

- [QuestFactory.sol:165](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/QuestFactory.sol#L165)
- [RabbitHoleReceipt.sol:71](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleReceipt.sol#L71)
- [RabbitHoleTickets.sol:60](https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/RabbitHoleTickets.sol#L60)

#### Recommended Mitigation Steps

Consider adding two step procedure on the critical functions where the first is announcing a pending address and the new address should then claim its role.

## [NC-18] Generate perfect code headers every time

#### Description

I recommend using header for Solidity code layout and readability.

> Ref: https://github.com/transmissions11/headers

#### Lines of code 

- [All Contracts](https://github.com/rabbitholegg/quest-protocol/tree/main/contracts)

#### Recommended Mitigation Steps

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## [NC-19] There is no need to cast a variable that is already an address, such as `address(x)`

#### Description

There is no need to cast a variable that is already an `address`, such as `address(x)`, `x` is also `address`.

```solidity
Erc1155Quest newQuest = new Erc1155Quest(
                rewardTokenAddress_,
                endTime_,
                startTime_,
                totalParticipants_,
                rewardAmountOrTokenId_,
                questId_,
                address(rabbitholeReceiptContract)
            );
```

#### Lines of code 

- [QuestFactory.sol:82](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L82)
- [QuestFactory.sol:115](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L115)

#### Recommended Mitigation Steps

Use directly variable :

```solidity
Erc1155Quest newQuest = new Erc1155Quest(
                rewardTokenAddress_,
                endTime_,
                startTime_,
                totalParticipants_,
                rewardAmountOrTokenId_,
                questId_,
                rabbitholeReceiptContract
            );
```

## [NC-20] Add NatSpec comment to `mapping`   

#### Description

Add NatSpec comments describing mapping keys and values.

```solidity
    mapping(uint => uint) public timestampForTokenId;
```

#### Lines of code 

- [Quest.sol:24](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L24)
- [QuestFactory.sol:20](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L20)
- [QuestFactory.sol:28](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L28)
- [QuestFactory.sol:30](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L30)
- [RabbitHoleReceipt.sol:30](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L30)
- [RabbitHoleReceipt.sol:34](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L34)

#### Recommended Mitigation Steps

```solidity
 /// @dev  uint(timestamp) => uint(token Id)
    mapping(uint => uint) public timestampForTokenId;
```
