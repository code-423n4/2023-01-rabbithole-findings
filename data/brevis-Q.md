# Incorrect Receipt Text Rendering

## Lines of Code

[https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L100](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L100)

## Vulnerability Details

### Impact

The receipt text is rendered incorrectly.

### Proof of Concept

Here is the code of `generateSVG` function (link provided in the **Lines of code** section above):

```javascript
// ReceiptRenderer.sol

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

In order to obtain a clean `svg` format, the params passed to `abi.encodePacked` can be concatenated into a single string by removing `'` and `,` characters as follows:

```bash
<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">
	<style>.base { fill: white; font-family: serif; font-size: 14px; }</style>
	<rect width="100%" height="100%" fill="black" />
	<text x="50%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest #questId_</text>
	<text x="70%" y="40%" class="base" dominant-baseline="middle" text-anchor="middle">RabbitHole Quest Receipt #tokenId_</text>
</svg>
```

*where again the variables `questId_` and `tokenId_` are placeholders for the actual values.*

Then, to check how it actually renders, an online SVG viewer like [https://www.svgviewer.dev/](https://www.svgviewer.dev/) can be utilized.

Here is the respective screenshot:

![Incorrect](https://res.cloudinary.com/enggym/image/upload/v1674806699/C4/Incorrect.jpg)

As can be seen above, the receipt (which is generated on the right hand side), contains two overlapping lines of text.

### Tools Used

Manual review.

Online SVG viewer [https://www.svgviewer.dev/](https://www.svgviewer.dev/).

### Recommended Mitigation Steps

To rectify that, the `x` and `y` attributes of the `text` element shall be modified accordingly. See below a sample proposed modification (the actual values shall be advised by the designer / project team):

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

```javascript
// ReceiptRenderer.sol

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
  

# Other Findings

## 1. Using Floating Pragma Version

### Vulnerability Details

Floating pragmas are used in all the contracts. However, the contracts should be deployed with the same compiler version. Locking the pragma helps ensuring that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively. 

Reference: [https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)

### **Recommended Mitigation Steps**

Lock the pragma version: delete `pragma solidity ^0.8.15` in favor of `pragma solidity 0.8.15`.   


## 2. Using `public` function visibility, where `private` could suffice

### Lines of Code

[https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L40](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L40)

### Vulnerability Details

The function `generateDataURI` is called only inside the same contract `ReceiptRenderer.sol`, therefore, there is no need to assign `public` visibility to it, which has higher gas consumption than the `private` one.  


## 3. Missing NatSpec

### Lines of Code

[https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L40](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L40)

### Vulnerability Details

The function `generateDataURI` doesn’t have NatSpec.  


## 4. Event is Missing `indexed` Field

### Lines of Code

[https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L8](https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/interfaces/IQuest.sol#L8)

### Vulnerability Details

The `event` fields are recommended to be `indexed`.