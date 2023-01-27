# Incorrect Receipt Rendering

The function [`generateSVG`](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L100) in `ReceiptRenderer.sol` contract contains erroneous code leading to an incorrect receipt rendering.

Here is the code:

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

It generates an `svg` image which will be subsequently part of the receipt, displaying the following two lines of information:

---

RabbitHole Quest #`questId_` 

RabbitHole Quest Receipt #`tokenId_` 

---

*where the variables `questId_` and `tokenId_` are placeholders for the actual values.*

In order to obtain the final `svg` code generated after packing the params passed to `abi.encodePacked` and afterwards respectively unpacking the output, we can concatenate these params into a single string by removing `'` and `,` characters thus obtaining the output `svg` code:

```bash
<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">
	<style>.base { fill: white; font-family: serif; font-size: 14px; }</style>
	<rect width="100%" height="100%" fill="black" />
	<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest #questId_</text>
	<text x="70%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #tokenId_</text>
</svg>
```

*where again the variables `questId_` and `tokenId_` are placeholders for the actual values.*

Then, to check how it renders, we can utilize an online SVG viewer like [https://www.svgviewer.dev/](https://www.svgviewer.dev/).

Here is the respective screenshot:

![Incorrect](https://res.cloudinary.com/enggym/image/upload/v1674806699/C4/Incorrect.jpg)

As can be seen above, the receipt is generated on the right, however, its two lines of content are overlapping.

To rectify that, we’ll have to tweak the `x` and `y` attributes of the `text` element as follows (the actual values shall be approved by the designer):

```bash
<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">
	<style>.base { fill: white; font-family: serif; font-size: 14px; }</style>
	<rect width="100%" height="100%" fill="black" />
	<text x="50%" y="45%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest #questId_</text>
	<text x="50%" y="55%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #tokenId_</text>
</svg>
```

As a result, the above code renders as:

![Correct](https://res.cloudinary.com/enggym/image/upload/v1674807280/C4/Correct.jpg)

Accordingly, to achieve the output displayed above, the `generateSVG` function shall be modified as shown below:

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
	// '<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest #',
        // questId_,
        // '</text>',
        '<text x="50%" y="45%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest #',
        questId_,
        '</text>',
	// '<text x="70%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #',
        // tokenId_,
        // '</text>',
        '<text x="50%" y="55%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #',
        tokenId_,
        '</text>',
        '</svg>'
    );
    return string(abi.encodePacked('data:image/svg+xml;base64,', Base64.encode(svg)));
}
```

Regards.