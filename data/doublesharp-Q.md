# The RoyaltyFeeSet event should not index the value

The royaltyFee should not be indexed in the RabbitHoleReceipt.RoyaltyFeeSet and RabbitHoleTickets.RoyaltyFeeSet events.

`event RoyaltyFeeSet(uint256 royaltyFee);`

# Quest contract should be abstract.

The Quest contract will revert if the `_calculateRewards()` and `_transferRewards()` are not implemented. It would be better to make the Quest contract abstract
and not provide implementations for the aforementioned functions.

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
