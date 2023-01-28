- Quest struct can be optimized to fit questAddress, totalParticipants, and numberMinted in 1 storage slot, to result in the following:
```
    struct Quest {
        mapping(address => bool) addressMinted;
        address questAddress;
        uint48 totalParticipants;
        uint48 numberMinted;
    }
```