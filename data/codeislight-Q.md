- use contractType enum instead of string memory, to help reduce any ambiguity. as an example the following enum can be used:
```
enum TOKEN_TYPE {ERC20,ERC1155};
``` 