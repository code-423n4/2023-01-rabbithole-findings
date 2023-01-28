### [NC-01] UNUSED VARIABLES

**contracts\RabbitHoleTickets.sol**

    109: function royaltyInfo(
        uint256 tokenId_, // Unused variable
        uint256 salePrice_
    ) external view override returns (address receiver, uint256 royaltyAmount) {
        uint256 royaltyPayment = (salePrice_ * royaltyFee) / 10_000;
        return (royaltyRecipient, royaltyPayment);
    }

**contracts\Quest.sol**

    123: function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {
        revert MustImplementInChild();
    }

    130: function _transferRewards(uint256 amount_) internal virtual {
        revert MustImplementInChild();
    }

### [NC-02] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USED TO IMMUTABLE RATHER THAN CONSTANT

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.
While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.
Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

**contracts\QuestFactory.sol**

    17: bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');


### [L-01] UNSPECIFIC COMPILER VERSION PRAGMA

Avoid floating pragmas for non-library contracts.
While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.
A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.
It is recommended to pin to a concrete compiler version.

    contracts\QuestFactory.sol = >  pragma solidity ^0.8.15;
    contracts\RabbitHoleReceipt.sol = > pragma solidity ^0.8.15;
    contracts\RabbitHoleReceipt.sol = >  pragma solidity ^0.8.15;
    contracts\Quest.sol = >  pragma solidity ^0.8.15;
    contracts\RabbitHoleTickets.sol = >  pragma solidity ^0.8.15;
    contracts\Erc20Quest.sol = >  pragma solidity ^0.8.15;
    contracts\Erc1155Quest.sol = >  pragma solidity ^0.8.15;
    contracts\ReceiptRenderer.sol = >  pragma solidity ^0.8.15;

### [L-02] – Unreachable code In function claim() in **contracts\Quest.sol**

    112: uint256 totalRedeemableRewards = _calculateRewards(redeemableTokenCount); // **always reverts. **

    123: function _calculateRewards(uint256 redeemableTokenCount_) internal virtual returns (uint256) {
        revert MustImplementInChild();
    }

     130: function _transferRewards(uint256 amount_) internal virtual {
        revert MustImplementInChild();
    }

Seems like it’s a bug from development because we can find normal functions without reverts: `function _transferRewards()` and `function _calculateRewards()` in `Erc20Quest` and `Erc1155Quest` contracts. 

### [L-03] Lack of zero address validation in constructors and initializers. 

Parameters used in constructor and initializer are used to initialize the state variable, errors in these can lead to the redeployment of contract

**contracts\Quest.sol**

    40: rewardToken = rewardTokenAddress_;

**contracts\QuestFactory.sol**

    45: claimSignerAddress = claimSignerAddress_; 
        rabbitholeReceiptContract = RabbitHoleReceipt(rabbitholeReceiptContract_);
        setProtocolFeeRecipient(protocolFeeRecipient_);

**contracts\RabbitHoleReceipt.sol**

    52: royaltyRecipient = royaltyRecipient_;
        minterAddress = minterAddress_;
        royaltyFee = royaltyFee_;
        ReceiptRendererContract = ReceiptRenderer(receiptRenderer_);

**contracts\RabbitHoleTickets.sol**

    41: royaltyRecipient = royaltyRecipient_;
        minterAddress = minterAddress_;
        royaltyFee = royaltyFee_;
        TicketRendererContract = TicketRenderer(ticketRenderer_);

**contracts\Erc20Quest.sol**

    39: protocolFeeRecipient = protocolFeeRecipient_;


### [L-04] Withdrawal functions don't check zero address values. 

Accidental input of address(0) can lead to loss of funds sending assets to a 0 address.

**contracts\Erc20Quest.sol **

    81: function withdrawRemainingTokens(address to_) public override onlyOwner {
        super.withdrawRemainingTokens(to_); 

        uint unclaimedTokens = (receiptRedeemers() - redeemedTokens) * rewardAmountInWeiOrTokenId;
        uint256 nonClaimableTokens = IERC20(rewardToken).balanceOf(address(this)) - protocolFee() - unclaimedTokens;
        IERC20(rewardToken).safeTransfer(to_, nonClaimableTokens);
    }

**contracts\Erc1155Quest.sol**

    54: function withdrawRemainingTokens(address to_) public override onlyOwner {
        super.withdrawRemainingTokens(to_);
        IERC1155(rewardToken).safeTransferFrom(
            address(this),
            to_,
            rewardAmountInWeiOrTokenId,
            IERC1155(rewardToken).balanceOf(address(this), rewardAmountInWeiOrTokenId),
            '0x00'
        );
    }
