## Struct packing

Some struct can be packed tighter to save storage slot, for example:

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L19-L24
```
    struct Quest {
        mapping(address => bool) addressMinted;
        address questAddress;
        uint totalParticipants;
        uint numberMinted;
    }
```

can be packed into

```
    struct Quest {
        mapping(address => bool) addressMinted;
        address questAddress;
        uint48 totalParticipants;
        uint48 numberMinted;
    }
```

To save 2 storage slot, while still allow 2^48 participant and reward to be minted which should be reasonably enough.

## Deploy with proxy

Quest contracts are deployed directly from the factory, which can be expensive. Consider deploying proxy to a shared Quest implementation to reduce deployment gas cost. Although this will increase each call to the contract slightly.