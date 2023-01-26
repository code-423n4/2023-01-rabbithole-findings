# QA report

## Low Risk
| L-N    |Issue|
|:------:|:----|
| [L&#x2011;01] | _safeMint is vulnerable to reentrancy | 1 |
| [L&#x2011;02] | Single-step ownership transfer pattern is dangerous | 1 |
| [L&#x2011;03] | Missing zero address check in `withdrawRemainingTokens()` | 2 |

Total: 3 issues

## Non-critical

| N-N    |Issue|
|:------:|:----|
| [N&#x2011;01] | Use latest Solidity version with a stable pragma statement | 2 |
| [N&#x2011;02] | Constants should be defined rather than using magic numbers  | 6 |
| [N&#x2011;03] | Missing event emission on setter functions in `QuestFactory` | 11 |
| [N&#x2011;04] | Typo in the comments| 19 |

Total: 4 issues

## Low Risk

### [L-01] _safeMint is vulnerable to reentrancy

`RabbitHoleReceipt:mint`

```solidity
function mint(address to_, string memory questId_) public onlyMinter {
        _tokenIds.increment();
        uint newTokenID = _tokenIds.current();
        questIdForTokenId[newTokenID] = questId_;
        timestampForTokenId[newTokenID] = block.timestamp;
        _safeMint(to_, newTokenID);
    }
```

When minting NFTs, the code uses _safeMint function of the OZ reference implementation. This function is safe because it checks whether the receiver can receive ERC721 tokens. The can prevent the case that a NFT will be minted to a contract that cannot handle ERC721 tokens. However, this external function call creates a security loophole. Specifically, the attacker can perform a reentrant call inside the onERC721Received callback. More detailed information why a reeentrancy can occur - https://blocksecteam.medium.com/when-safemint-becomes-unsafe-lessons-from-the-hypebears-security-incident-2965209bda2a.

Use the nonReentrant modifier in the functions where you use _safeMint.

### [L-02] Single-step ownership transfer pattern is dangerous

Inheriting from OpenZeppelin's `Ownable` contract means you are using a single-step ownership transfer pattern. If an admin provides an incorrect address for the new owner this will result in none of the `onlyOwner` marked methods being callable again. The better way to do this is to use a two-step ownership transfer approach, where the new owner should first claim its new rights before they are transferred.

Use OpenZeppelin's `Ownable2Step` instead of `Ownable`

### [L-03] Missing zero address check in `withdrawRemainingTokens()`

File: `Erc1155Quest.sol`

Checking addresses against zero-address during initialization or during setting is a security best-practice. The check is also missing in the constructor for param `rewardTokenAddress_` 

```solidity
function withdrawRemainingTokens(address to_) public override onlyOwner {
        super.withdrawRemainingTokens(to_);
        IERC1155(rewardToken).safeTransferFrom(
            address(this),
            to_,
            rewardAmountInWeiOrTokenId,
            IERC1155(rewardToken).balanceOf(address(this), rewardAmountInWeiOrTokenId),
            '0x00'
        );
    }
```

Add zero address checks.

## Non-critical

### [NC-01] Use latest Solidity version with a stable pragma statement
Using a floating pragma ^0.8.15 statement is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode. Also use latest Solidity version to get all compiler features, bugfixes and optimizations

### [NC-02] Constants should be defined rather than using magic numbers 

File: `Erc20Quest.sol`

```solidity
return (maxTotalRewards() * questFee) / 10_000;
```

```solidity
return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
```


### [NC-03] Missing event emission on setter functions in `QuestFactory`

 Setters should emit an event so that Dapps can detect important changes to storage. Some of the setter functions in `RabbitHoleReceipt.sol` are missing event emission as well.

Example:

```solidity
function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
        claimSignerAddress = claimSignerAddress_;
    }
```

### [NC-04] Typo in the comments

File: `QuestFactory.sol`

`remave` -> `remove`
