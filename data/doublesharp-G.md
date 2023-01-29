# Use a Bitmaps.Bitmap for Quest.claimedList.

The `claimedList` is a mapping of boolean values which will take up on storage slot per entry. As the key is the incremental token ID of RabbitHoleReciept a large gas savings for storage can be realized by using a bitmap. Convert the variable from `mapping(uint256 => bool)` to `BitMaps.BitMap`

```solidity
import { BitMaps } from '@openzeppelin/contracts/utils/structs/BitMaps.sol';

contract Quest is Ownable, IQuest {
  using BitMaps for BitMaps.BitMap;
  BitMaps.BitMap private claimedList;

  function _setClaimed(uint256[] memory tokenIds_) private {
    for (uint i = 0; i < tokenIds_.length; i++) {
      claimedList.set(tokenIds_[i]);
    }
  }

  function isClaimed(uint256 tokenId_) public view returns (bool) {
    return claimedList.get(tokenId_);
  }
}
```

Update test to claim for two tokens

```
describe('Erc1155Quest', () => {
  describe('claim()', async () => {
    it('should only transfer the correct amount of rewards', async () => {
      await deployedRabbitholeReceiptContract.mint(owner.address, questId)
      await deployedQuestContract.start()

      await ethers.provider.send('evm_increaseTime', [86400])

      expect(await deployedSampleErc1155Contract.balanceOf(owner.address, rewardAmount)).to.equal(0)

      const totalTokens = await deployedRabbitholeReceiptContract.getOwnedTokenIdsOfQuest(questId, owner.address)
      expect(totalTokens.length).to.equal(1)

      expect(await deployedQuestContract.isClaimed(1)).to.equal(false)

      await deployedQuestContract.claim()
      expect(await deployedSampleErc1155Contract.balanceOf(owner.address, rewardAmount)).to.equal(1)

      // x 2
      await deployedRabbitholeReceiptContract.mint(owner.address, questId)
      await ethers.provider.send('evm_increaseTime', [86400])
      await deployedQuestContract.claim()

      await ethers.provider.send('evm_increaseTime', [-86400 * 2])
    })
  })
})
```

Before

```
·---------------------------------------------------|---------------------------|--------------|-----------------------------·
|  Contract           ·  Method                     ·  Min        ·  Max        ·  Avg         ·  # calls      ·  eur (avg)  │
······················|·····························|·············|·············|··············|···············|··············
|  Erc1155Quest       ·  claim                      ·          -  ·          -  ·      127666  ·            1  ·          -  │
······················|·····························|·············|·············|··············|···············|··············
|  Erc1155Quest x 2   ·  claim                      ·     102350  ·     127666  ·      118887  ·            2  ·          -  │
······················|·····························|·············|·············|··············|···············|··············
|  Erc1155Quest x 3   ·  claim                      ·     102350  ·     127666  ·      113750  ·            3  ·          -  │
```

After

```
·---------------------------------------------------|---------------------------|--------------|-----------------------------·
|  Contract           ·  Method                     ·  Min        ·  Max        ·  Avg         ·  # calls      ·  eur (avg)  │
······················|·····························|·············|·············|··············|···············|··············
|  Erc1155Quest       ·  claim                      ·          -  ·          -  ·      127744  ·            1  ·          -  │
······················|·····························|·············|·············|··············|···············|··············
|  Erc1155Quest x 2   ·  claim                      ·      83406  ·     127744  ·      112620  ·            2  ·          -  │
······················|·····························|·············|·············|··············|···············|··············
|  Erc1155Quest x 3   ·  claim                      ·      83406  ·     127744  ·      100506  ·            3  ·          -  │
```

# Decrement loops to take advantage of ISZERO check

By initializing the iterator to the loop bounds you can decrement the iterator and check for `iter != 0` to take advantage of `ISZERO` to save gas when the order doesn't matter.

Ideally combined with using a `BitMaps.BitMap`.

Quest.sol:69

```
function _setClaimed(uint256[] memory tokenIds_) private {
    for (uint i = tokenIds_.length; i != 0;) {
        unchecked{ --i; }
        claimedList[tokenIds_[i]] = true;
    }
}
```

Quest.sol:103

```
uint256 redeemableTokenCount = 0;
for (uint i = tokens.length; i != 0;) {
    unchecked{ --i; }
    if (!isClaimed(tokens[i])) {
        unchecked{ ++redeemableTokenCount; }
    }
}
```

RabbitHoleReceipt.sol:117

```
for (uint i = msgSenderBalance; i != 0;) {
    unchecked{ --i; }
    uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
    if (keccak256(bytes(questIdForTokenId[tokenId])) == keccak256(bytes(questId_))) {
        tokenIdsForQuest[i] = tokenId;
        unchecked{ ++foundTokens; }
    }
}
```

# Calculate questId hash outside of loop in RabbitHoleReceipt.getOwnedTokenIdsOfQuest()

The quest ID hash `keccak256(bytes(questId_))` is calculated on every loop. Creating once outside the loop will save gas.

```
bytes32 questIdHash = keccak256(bytes(questId_));
for (uint i = 0; i < msgSenderBalance; i++) {
    uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
    if (keccak256(bytes(questIdForTokenId[tokenId])) == questIdHash) {
        tokenIdsForQuest[i] = tokenId;
        foundTokens++;
    }
}
```

# Use assembly instead of loop in RabbitHoleReceipt.getOwnedTokenIdsOfQuest()

Save gas by modifying the returned array length in solidty vs doing a second loop to save gas. Test show gas used of 39012 before and 38470 after for the test case.

```
  it('should check the gas for getOwnedTokenIdsOfQuest', async () => {
    await deployedRabbitholeReceiptContract.mint(owner.address, questId)
    await deployedQuestContract.start()

    await ethers.provider.send('evm_increaseTime', [86400])

    expect(await deployedSampleErc1155Contract.balanceOf(owner.address, rewardAmount)).to.equal(0)

    const gasTotalTokensGas = await deployedRabbitholeReceiptContract.estimateGas.getOwnedTokenIdsOfQuest(
      questId,
      owner.address
    )
    console.log('gasTotalTokensGas', gasTotalTokensGas.toString()) // 39012 -> 38470
  await ethers.provider.send('evm_increaseTime', [-86400])
  })
```

RabbitHoleReceipt.sol:109

```
    function getOwnedTokenIdsOfQuest(
        string memory questId_,
        address claimingAddress_
    ) public view returns (uint[] memory) {
        uint msgSenderBalance = balanceOf(claimingAddress_);
        uint[] memory tokenIdsForQuest = new uint[](msgSenderBalance);
        uint foundTokens = 0;

        bytes32 questIdHash = keccak256(bytes(questId_));
        for (uint i = msgSenderBalance; i != 0; ) {
            unchecked { --i; }
            uint tokenId = tokenOfOwnerByIndex(claimingAddress_, i);
            if (keccak256(bytes(questIdForTokenId[tokenId])) == questIdHash) {
                tokenIdsForQuest[i] = tokenId;
                unchecked { ++foundTokens; }
            }
        }

        // truncate the array if we found less than the balance
        if (foundTokens != msgSenderBalance) {
            assembly {
                mstore(tokenIdsForQuest, foundTokens)
            }
        }

        return tokenIdsForQuest;
    }
```

# Pack storage slots by making royaltyFee a uint16 in RabbitHoleReceipt and RabbitHoleTickets

The royalty is using a divisor of 10000 basis points so a uint16 will hold the max value and allow the variable to be packed with `address public minterAddress`

RabbitHoleReceipt.sol

```
    // storage
    mapping(uint => string) public questIdForTokenId;
    address public royaltyRecipient;
    address public minterAddress;
    uint16 public royaltyFee;
    // ...

    function initialize(
        address receiptRenderer_,
        address royaltyRecipient_,
        address minterAddress_,
        uint16 royaltyFee_
    ) public initializer {
      //...
    }

    function setRoyaltyFee(uint16 royaltyFee_) external onlyOwner {
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
```

RabbitHoleTickets.sol

```
    // storage
    address public royaltyRecipient;
    address public minterAddress;
    uint16 public royaltyFee;

    function initialize(
        address ticketRenderer_,
        address royaltyRecipient_,
        address minterAddress_,
        uint16 royaltyFee_
    ) public initializer {
      // ...
    }

    function setRoyaltyFee(uint16 royaltyFee_) public onlyOwner {
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
```

# Use constants for unchanging hash values in QuestFactory

The checks on QuestFactory.sol:72 and :105 should be made constant.

```
bytes32 private constant QUEST_ERC20 = keccak256(abi.encodePacked('erc20'));
bytes32 private constant QUEST_ERC1155 = keccak256(abi.encodePacked('erc1155'));

if (keccak256(abi.encodePacked(contractType_)) == QUEST_ERC20) {
  // ...
}
if (keccak256(abi.encodePacked(contractType_)) == QUEST_ERC1155) {
  // ...
}
```

# QuestFactory questIdCount increments can be unchecked

It is not realistically possible to overflow the questIdCount on lines 101 and 132

```
unchecked { ++questIdCount; }
```

# QuestFactory storage packing

Using uint16 for the questFee will not overflow the max fee and allow it to be packed in storage slots when placed under the `protocolFeeRecipient`

```
    address public claimSignerAddress;
    address public protocolFeeRecipient;
    uint16 public questFee;

    function setQuestFee(uint16 questFee_) public onlyOwner {
        if (questFee_ > 10_000) revert QuestFeeTooHigh();
        questFee = questFee_;
    }
```

# Refactor QuestFactory.mintReceipt() to save gas

Use `storage` and memory variables to save gas when calling mintReceipt. Tests show 288053 gas before and 286687 gas after changes.

Convert check of `_quest.numberMinted + 1 > _quest.totalParticipants` to `_quest.numberMinted >= _quest.totalParticipants` to prevent addition.

Unchecked increment of `_quest.numberMinted` as it is not realistic to overlow a uint256.

```
    function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) external {
        Quest storage _quest = quests[questId_];
        address _sender = msg.sender;

        if (_quest.numberMinted >= _quest.totalParticipants) revert OverMaxAllowedToMint();
        if (_quest.addressMinted[_sender] == true) revert AddressAlreadyMinted();
        if (keccak256(abi.encodePacked(_sender, questId_)) != hash_) revert InvalidHash();
        if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();

        _quest.addressMinted[_sender] = true;
        unchecked{ _quest.numberMinted = ++_quest.numberMinted; }
        emit ReceiptMinted(_sender, questId_);
        rabbitholeReceiptContract.mint(_sender, questId_);
    }
```

# The RoyaltyFeeSet event should not index the value

The royaltyFee should not be indexed in the RabbitHoleReceipt.RoyaltyFeeSet and RabbitHoleTickets.RoyaltyFeeSet events.

`event RoyaltyFeeSet(uint256 royaltyFee);`

# Quest contract should be abstract.

The Quest contract will revert if the `_calculateRewards()` and `_transferRewards()` are not implemented. It would be better to make the Quest contract abstract
and not provide implementations for the aformentioned functions.

```
abstract contract Quest is Ownable, IQuest {
    /// @notice Calculate the amount of rewards
    /// @dev This function must be implemented in the child contracts
    function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256);

    /// @notice Transfer the rewards to the user
    /// @dev This function must be implemented in the child contracts
    /// @param amount_ The amount of rewards to transfer
    function _transferRewards(uint256 amount_) internal virtual;
}
```
