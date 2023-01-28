- Quest struct can be optimized to fit questAddress, totalParticipants, and numberMinted in 1 storage slot, to result in the following:
```
    struct Quest {
        mapping(address => bool) addressMinted;
        address questAddress;
        uint48 totalParticipants;
        uint48 numberMinted;
    }
```
- use contractType enum instead of string memory, to help reduce any ambiguity and save gas on keccak256 and abi.encodePacked. as an example the following enum can be used:
```
enum TOKEN_TYPE {ERC20,ERC1155};
``` 
- questId variable value doesn't change in Quest contract, advised to be defined as immutable to save on gas.  