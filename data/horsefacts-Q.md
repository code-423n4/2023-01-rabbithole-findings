## Low

### `Quest`: Consider burning tokens on claim

The Rabbithole team and docs mention that receipt tokens may circulate on the secondary market. However, this means already claimed receipt tokens may circulate and be listed on secondary markets alongside unclaimed receipt tokens. Additionally, receipts may be flash loaned and claimed if they are deposited in certain protocols. Consider burning the user's receipt token when a claim is made or updating its onchain metadata with a clear indicator that rewards have already been claimed to mitigate secondary market scams related to receipt tokens.

### `Quest`: Reentrancy in `claim`

[`Quest#claim`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96) increments `redeemedTokens` (an effect), after calling `_transferRewards` (an interaction). Since `_transferRewards` may `safeTransfer` an ERC-1155 token, this introduces a reentrancy opportunity:

```solidity
    /// @notice Allows user to claim the rewards entitled to them
    /// @dev User can claim based on the (unclaimed) number of tokens they own of the Quest
    function claim() public virtual onlyQuestActive {
        if (isPaused) revert QuestPaused();

        uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);

        if (tokens.length == 0) revert NoTokensToClaim();

        uint256 redeemableTokenCount = 0;
        for (uint i = 0; i < tokens.length; i++) {
            if (!isClaimed(tokens[i])) {
                redeemableTokenCount++;
            }
        }

        if (redeemableTokenCount == 0) revert AlreadyClaimed();

        uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);
        _setClaimed(tokens);
        _transferRewards(totalRedeemableRewards);
        redeemedTokens += redeemableTokenCount;

        emit Claimed(msg.sender, totalRedeemableRewards);
    }
```

The impact here looks limited, since `redeemedTokens` is not used internally, but it's a good practice to follow checks-effects-interactions and make the state change before any token transfers:

```solidity
    /// @notice Allows user to claim the rewards entitled to them
    /// @dev User can claim based on the (unclaimed) number of tokens they own of the Quest
    function claim() public virtual onlyQuestActive {
        if (isPaused) revert QuestPaused();

        uint[] memory tokens = rabbitHoleReceiptContract.getOwnedTokenIdsOfQuest(questId, msg.sender);

        if (tokens.length == 0) revert NoTokensToClaim();

        uint256 redeemableTokenCount = 0;
        for (uint i = 0; i < tokens.length; i++) {
            if (!isClaimed(tokens[i])) {
                redeemableTokenCount++;
            }
        }

        if (redeemableTokenCount == 0) revert AlreadyClaimed();

        uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount);
        redeemedTokens += redeemableTokenCount;
        _setClaimed(tokens);
        _transferRewards(totalRedeemableRewards);

        emit Claimed(msg.sender, totalRedeemableRewards);
    }
```

### `RabbitHoleTickets`: No upper bound or validation of `royaltyFee`

There is no upper bound or validation of the admin-controlled `royaltyFee` parameter:

[`RabbitHoleTickets#setRoyaltyFee`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L64):

```solidity
    /// @dev set the royalty fee
    /// @param royaltyFee_ the royalty fee
    function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
        royaltyFee = royaltyFee_;
        emit RoyaltyFeeSet(royaltyFee_);
    }
```

Consider enforcing a reasonable upper limit on this parameter, or at least validating that the value is less than 100%. [EIP-2981](https://eips.ethereum.org/EIPS/eip-2981) does not specify how marketplaces should behave if `getRoyalties` returns a value greater than 100%, but it seems likely to break things.

### `ReceiptRenderer`: `tokenId_` is not converted to string

[`ReceiptRenderer#generateSVG`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L96-L114) does not convert its `tokenId_` arg to a string when constructing the SVG:

```solidity
    /// @dev generates the on-chain SVG for an ERC-721 token ID
    /// @param tokenId_ The token id to generate the svg for
    /// @param questId_ The questId tied to the tokenId
    /// @return encoded JSON for an SVG image
    function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {
        bytes memory svg = abi.encodePacked(
            '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
            '<style>.base { fill: white; font-family: serif; font-size: 14px; }</style>',
            '<rect width="100%" height="100%" fill="black" />',
            '<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest #',
            questId_,
            '</text>',
            '<text x="70%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #',
            tokenId_,
            '</text>',
            '</svg>'
        );
        return string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));
    }
```

This means the raw bytes of the `uint tokenID_` argument will be embedded in the SVG. For example, token Id `310939249775` will render `Rabbithole Quest Receipt #Hello` rather than `Rabbithole Quest Receipt #310939249775` as expected. Token IDs whose hex bytes include `0x3c` ('<'), `0x3e` ('>'), and other special or invalid characters may break the rendered SVG image.

Suggestion: Convert the numeric token ID to a string using `Strings.toString()`:

```solidity
    /// @dev generates the on-chain SVG for an ERC-721 token ID
    /// @param tokenId_ The token id to generate the svg for
    /// @param questId_ The questId tied to the tokenId
    /// @return encoded JSON for an SVG image
    function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {
        bytes memory svg = abi.encodePacked(
            '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
            '<style>.base { fill: white; font-family: serif; font-size: 14px; }</style>',
            '<rect width="100%" height="100%" fill="black" />',
            '<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest #',
            questId_,
            '</text>',
            '<text x="70%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #',
            tokenId_.toString(),
            '</text>',
            '</svg>'
        );
        return string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));
    }
```

Additionally, a technique I've found really helpful for developing onchain SVGs is creating a ["visual QA"](https://github.com/w1nt3r-eth/hot-chain-svg#visual-qa) script that renders and saves a batch of randomly generated token images to a temporary folder. This is a great way to catch small visual bugs in dynamic onchain SVGs.

### `ReceiptRenderer`: Overlapping text lines

The two text lines in [`ReceiptRenderer#generateSVG`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L96-L114) are placed at the same `y=40%` position and will overlap each other when rendered:

```solidity
    /// @dev generates the on-chain SVG for an ERC-721 token ID
    /// @param tokenId_ The token id to generate the svg for
    /// @param questId_ The questId tied to the tokenId
    /// @return encoded JSON for an SVG image
    function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {
        bytes memory svg = abi.encodePacked(
            '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">',
            '<style>.base { fill: white; font-family: serif; font-size: 14px; }</style>',
            '<rect width="100%" height="100%" fill="black" />',
            '<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest #',
            questId_,
            '</text>',
            '<text x="70%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #',
            tokenId_,
            '</text>',
            '</svg>'
        );
        return string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));
    }
```

Suggestion: Place the second line at a lower y position in the SVG:

```solidity
            '<text x="70%" y="60%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #',
```

(It looks like it might be possible that the x- and y- positions here are accidentally reversed. Again, I recommend a visual QA script to catch stuff like this that's difficult to test for otherwise).

### Emit events from admin functions

Throughout the codebase, there are a number of admin-owned functions that change configuration/state variables without emitting events. For example, [`RabbitHoleTickets#setTicketRenderer`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L52-L56):

```solidity
    /// @dev set the ticket renderer contract
    /// @param ticketRenderer_ the address of the ticket renderer contract
    function setTicketRenderer(address ticketRenderer_) public onlyOwner {
        TicketRendererContract = TicketRenderer(ticketRenderer_);
    }
```

Consider adding events to these functions logging the previous and new configuration values. It's easy to watch for these events using off chain monitoring tools like Forta or OZ Defender, which will alert on any unexpected changes. Additionally, it provides a hook for _anyone_ to monitor your contracts, which can build trust with your users.

Other functions:
- [`RabbitHoleTickets#setRoyaltyRecipient`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L60)
- [`RabbitHoleReceipt#setReceiptRenderer`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L65)
- [`RabbitHoleReceipt#setRoyaltyRecipient`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L71)
- [`RabbitHoleReceipt#setQuestFactory`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L77)
- [`QuestFactory#setClaimSignerAddress`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L159)
- [`QuestFactory#setProtocolFeeRecipient`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L165)
- [`QuestFactory#setRewardAllowlistAddress`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L179)
- [`QuestFactory#setQuestFee`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L186)
- [`Quest#start`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L50)
- [`Quest#pause`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57)
- [`Quest#unPause`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L63)

In the case of `pause` and `unPause`, consider emitting the same `Paused(address)` and `Unpaused(address)` events as OZ `Pausable` (or simply replace them with `Pausable`), since a number of off-chain tools automatically listen for these events.

### `QuestFactory`: Use EIP-712 typed structured signatures

Ethereum signed messages are not easily readable by end users, and potentially replayable across chains and contract implementations. None of these are critical concerns, but consider using an [EIP-712 typed signature](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712) with a structured format such as:

```
MintReceipt(address receiver,string questId)
```

### `QuestFactory`: Changing `claimSignerAddress` will invalidate previous signatures

If the owner of `QuestFactory` changes the `claimSignerAddress`, it will invalidate any previously issued signatures, even if they were issued by a previously valid signer. Make sure you account for this in the design of your claim workflow.

## Noncritical

### `ERC1155Quest`: Use empty string for `safeTransferFrom` data

Calling `IERC1155.safeTransferFrom` with `"0x00"` as the `data` argument will send the literal string `"0x00"` as `data`, i.e. the hex bytes `0x30783030`:

[`ERC1155Quest#_transferRewards`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L41):

```solidity
    /// @dev Transfers the reward token `rewardAmountInWeiOrTokenId` to the msg.sender
    /// @param amount_ The amount of reward tokens to transfer
    function _transferRewards(uint256 amount_) internal override {
        IERC1155(rewardToken).safeTransferFrom(address(this), msg.sender, rewardAmountInWeiOrTokenId, amount_, '0x00');
    }
```

If you want to send empty bytes, use an empty string instead:

```solidity
    /// @dev Transfers the reward token `rewardAmountInWeiOrTokenId` to the msg.sender
    /// @param amount_ The amount of reward tokens to transfer
    function _transferRewards(uint256 amount_) internal override {
        IERC1155(rewardToken).safeTransferFrom(address(this), msg.sender, rewardAmountInWeiOrTokenId, amount_, '');
    }
```

This should save a little gas as well.

### `ERC20Quest`: Extract basis points denominator to named constant

Consider extracting the magic number `10_000` to a named constant, like `BASIS_POINTS_DENOMINATOR`.

[`ERC20Quest#maxProtocolReward`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L49):

```solidity
    /// @notice Function that gets the maximum amount of rewards that can be claimed by the protocol or the quest deployer
    /// @dev The 10_000 comes from Basis Points: https://www.investopedia.com/terms/b/basispoint.asp
    /// @return The maximum amount of rewards that can be claimed by the protocol or the quest deployer
    function maxProtocolReward() public view returns (uint256) {
        return (maxTotalRewards() * questFee) / 10_000;
    }
```

### `QuestFactory`: Use OZ `toEthSignedMessageHash`

The OpenZeppelin ECDSA helper library (already imported in `QuestFactory`), includes a `toEthSignedMessageHash` helper function. Consider using it in [`QuestFactory#recoverSigner`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L210-L213) for more readable code:

Suggestion:

```solidity
        bytes32 messageDigest = ECDSAUpgradeable.toEthSignedMessageHash(hash_);
        return ECDSAUpgradeable.recover(messageDigest, signature_);
    }
```

Or, with a `using ECDSAUpgradeable for bytes32` statement at the top of your contract:

```solidity
        return hash_.toEthSignedMessageHash().recover(signature_);
    }
```

### `ReceiptRenderer`: Use OZ `Strings.toHexString`

The OpenZeppelin `Strings` helper includes a `toHexString(address)` [function](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/49c0e4370d0cc50ea6090709e3835a3091e33ee2/contracts/utils/Strings.sol#L67) that performs the same conversion as this line in[`ReceiptRenderer#generateDataURI`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L60):

```solidity
            generateAttribute('Reward Address', Strings.toHexString(uint160(rewardAddress_), 20)),
```

Consider using it for readability. Suggestion:

```solidity
            generateAttribute('Reward Address', rewardAddress_.toHexString()),
```

### `ReceiptRenderer` and `TicketRenderer`: Use `string.concat`

A `string.concat` function is available in Solidity 0.8.15, which is a little more concise than calling `string(abi.encodePacked(bytes))`.

For example, the following:

[`ReceiptRenderer#GenerateAttribute`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L79-L94):

```solidity
    /// @dev generates an attribute object for an ERC-721 token
    /// @param key The key for the attribute
    /// @param value The value for the attribute
    function generateAttribute(string memory key, string memory value) public pure returns (string memory) {
        bytes memory attribute = abi.encodePacked(
            '{',
            '"trait_type": "',
            key,
            '",',
            '"value": "',
            value,
            '"',
            '}'
        );
        return string(attribute);
    }
```

Can be written using `string.concat` as:

```solidity
    /// @dev generates an attribute object for an ERC-721 token
    /// @param key The key for the attribute
    /// @param value The value for the attribute
    function generateAttribute(string memory key, string memory value) public pure returns (string memory) {
        return string.concat(
            '{',
            '"trait_type": "',
            key,
            '",',
            '"value": "',
            value,
            '"',
            '}'
        );
    }
```

Anywhere you see the `string(abi.encodePacked(bytes))` idiom is a candidate for replacement with `string.concat`. A few suggestions:

- [`ReceiptRenderer#generateTokenURI`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L37)
- [`ReceiptRenderer#generateSVG`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L113)
- [`TicketRenderer#generateTokenURI`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L30)
- [`TicketRenderer#generateSVG`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L46)

### Add `contractURI()` to token contracts

Although it's not part of the ERC721 spec, you might consider adding OpenSea's `contractURI()` [function](https://docs.opensea.io/docs/contract-level-metadata) to your token contracts, which is used by OpenSea and other marketplaces when present.

### Review royalty enforcement

Although the Rabbithole contracts implement EIP-2981, enforcement will be optional on OpenSea. This may be an intentional design choice, but it's come as a surprise a few projects who expected EIP-2981 royalties to be honored. Consider whether or not you want to comply with their platform's recently added [royalty enforcement requirements](https://github.com/ProjectOpenSea/operator-filter-registry).

### `Quest`: Unused imports

`ECDSA` is unused in [`Quest.sol#L7`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L7):

```solidity
import {ECDSA} from '@openzeppelin/contracts/utils/cryptography/ECDSA.sol';
```

### `IQuest`: Open TODO and outdated comment:

There's an open TODO and outdated comment at the top of `IQUest.sol`:

```solidity
// TODO clean this whole thing up
// Allows anyone to claim a token if they exist in a merkle root.
```