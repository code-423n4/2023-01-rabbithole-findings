# 1. Incorrect Minter Address Validation in `Mint` Function

Link : https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L98

Summary: 

The `mint` function in the `RabbitHoleReceipt` contract does not correctly check the `msg.sender` address for minter permissions. The `onlyMinter` modifier is used to check if the `msg.sender` is equal to the contract's minter address, but the `msg.sender` variable is never updated within the modifier. As a result, the check will always return false and any address will be able to mint new tokens.

Impact: 

This vulnerability allows any address to mint new tokens without proper authorization. This could result in an attacker creating an arbitrary number of tokens and potentially reselling them. Additionally, the `questIdForTokenId` and `timestampForTokenId` mappings will be updated with invalid values, which may cause issues with later functions that rely on this data.

Recommendation: 

The `onlyMinter` modifier should be updated to use the `require` statement to check if the `msg.sender` address is equal to the minter address, rather than just a comparison. A sample fix would be:

```
modifier onlyMinter() {
    require(msg.sender == minterAddress, "Sender is not authorized minter");
    _;
}
```
Example:
```
modifier onlyMinter() {
    require(msg.sender == minterAddress, "Sender is not authorized minter");
    _;
}

function mint(address to_, string memory questId_) public onlyMinter {
    _tokenIds.increment();
    uint newTokenID = _tokenIds.current();
    questIdForTokenId[newTokenID] = questId_;
    timestampForTokenId[newTokenID] = block.timestamp;
    _safeMint(to_, newTokenID);
}
```

# 2. Unbounded Array Vulnerability in `Claim` Function

Link : https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96

Summary: 

The `claim` function in the Quest contract has an unbounded array vulnerability that could lead to an Out-of-Gas (OOG) error and make the contract unresponsive. The vulnerability occurs because the `_setClaimed` function is being called with an arbitrarily long `tokenIds_` array, which could consume all the gas in the block, causing the transaction to fail.

Impact: 

An attacker could submit a malicious transaction that contains a long array, causing the claim function to run out of gas, and making the contract unresponsive. This could result in a Denial-of-Service (DoS) attack on the contract.

Recommendation: 
To prevent this vulnerability, the size of the `tokenIds_` array should be limited to a reasonable number of items. For example, you could limit the length of the `tokenIds_` array to `256` items. Additionally, you should perform a gas check before calling the `_setClaimed` function to ensure that there is enough gas available to complete the transaction.

Example: 
```
// Limit the size of the tokenIds_ array to 256 items
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

    // Check if there is enough gas to complete the transaction
    require(redeemableTokenCount <= 256, "Array size is too large");

    uint256[] memory tokenIds = new uint256[](redeemableTokenCount);
    uint256 j = 0;
    for (uint i = 0; i < tokens.length; i++) {
        if (!isClaimed(tokens[i])) {
            tokenIds[j] = tokens[i];
            j++;
        }
    }

    _setClaimed(tokenIds);
}
```

# 3. Missing `require` condition in function `start`

Link : https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L58

Summary:

The function `start` doesn't have a require condition to check if the current time is greater than or equal to the start time of the quest.

Impact: 

If a user calls `start` before the quest's start time, the quest may be started but the rewards will not be redeemable by the users.

Recommendation: 

Add a `require` condition to check if the current time is greater than or equal to the start time of the quest.

Example:

```
function start() public override {
    require(now >= startTime, "Start time has not been reached yet.");
    if (IERC20(rewardToken).balanceOf(address(this)) < maxTotalRewards() + maxProtocolReward())
        revert TotalAmountExceedsBalance();
    super.start();
}
```

# 4. ERC1155 Quest Contract Incorrectly Reverts When Insufficient Tokens

Link : https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L33

Summary: 

The `start()` function of the `Erc1155Quest` contract reverts when the balance of the specified reward token is less than the number of total participants, even if the contract is only transferring one token to each participant.

Impact: 

If the balance of the reward token is less than the number of total participants, the `start()` function will revert and the quest cannot be started, effectively locking up the reward token. This can lead to loss of funds for the contract owner and participants.

Recommendation: 

The `start()` function should be updated to check if the balance is less than the number of tokens being transferred, not the number of total participants.

Example: 

If the contract has 100 participants and the reward token balance is 1 token, the `start()` function will currently revert and the quest cannot be started. The function should be updated to check if the balance is less than 1, not 100.

# 5. Unchecked balance before transfer in Erc1155Quest contract

Link: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L41

Summary: 

The `start` function of the Erc1155Quest contract is checking the balance of the reward token before starting the quest, but it is not checking the balance of the recipient before transferring the tokens.

Impact: 

If the recipient's balance is lower than the transferred tokens, the transfer will fail and the tokens will be lost forever.

Recommendation: Add a check for the balance of the recipient before the transfer in the `_transferRewards` function. An example implementation would be:

```
function _transferRewards(uint256 amount_) internal override {
    require(IERC1155(rewardToken).balanceOf(msg.sender) >= amount_, "Recipient balance is lower than the transferred amount");
    IERC1155(rewardToken).safeTransferFrom(address(this), msg.sender, rewardAmountInWeiOrTokenId, amount_, '0x00');
}
```

# 6. Insecure Data URI Generation in Contract `ReceiptRenderer`

Link : https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L21

Summary: 

The contract `ReceiptRenderer` generates token URI based on user-supplied information using the function `generateTokenURI()`. This information is included in the encoded JSON data and can be altered by an attacker to include malicious data.

Impact: 

An attacker could potentially exploit this vulnerability to inject malicious data into the encoded JSON and potentially execute arbitrary code.

Recommendation:

•	Validate user input to ensure that it does not include malicious data before including it in the generated token URI.
•	Use a library such as the OWASP Anti-Samy project to sanitize user input before including it in the generated token URI.

Example: The following is an example of malicious user input that could be included in the generated token URI:

```
function generateTokenURI(
        uint tokenId_,
        string memory questId_,
        uint totalParticipants_,
        bool claimed_,
        uint rewardAmount_,
        address rewardAddress_
    ) public view virtual returns (string memory) {
        bytes memory dataURI = generateDataURI(
            tokenId_,
            "\u200Bjavascript:alert(1)",
            totalParticipants_,
            claimed_,
            rewardAmount_,
            rewardAddress_
        );
        return string(abi.encodePacked('data:application/json;base64,', Base64.encode(dataURI)));
    }
```
# 7. Inadequate Input Validation in `generateDataURI` function

Link : https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L40

Summary: 

The function `generateDataURI` does not validate the input parameters and may allow for malicious users to inject unwanted data into the returned string.

Impact: 

An attacker could inject malicious content into the string returned by `generateDataURI`, potentially leading to code execution or unauthorized actions on behalf of the contract.

Recommendation: 

Input validation should be added to the `generateDataURI` function to ensure that all inputs are of the correct type and format. An example of a validator function could look like this:

```
function validateInputs(
    uint tokenId_,
    string memory questId_,
    uint totalParticipants_,
    bool claimed_,
    uint rewardAmount_,
    address rewardAddress_
) internal view returns (bool) {
    if (tokenId_ == 0) {
        return false;
    }
    if (questId_.length == 0) {
        return false;
    }
    if (totalParticipants_ == 0) {
        return false;
    }
    if (!isValidAddress(rewardAddress_)) {
        return false;
    }
    return true;
}

function generateDataURI(
    uint tokenId_,
    string memory questId_,
    uint totalParticipants_,
    bool claimed_,
    uint rewardAmount_,
    address rewardAddress_
) public view virtual returns (bytes memory) {
    require(validateInputs(tokenId_, questId_, totalParticipants_, claimed_, rewardAmount_, rewardAddress_), "Invalid inputs");
    // Rest of the function...
}
```
