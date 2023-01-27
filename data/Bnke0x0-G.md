

### [G01] State variables only set in the constructor should be declared `immutable`

#### Impact
Avoids a Gusset (20000 gas)
#### Findings:
```
quest-protocol/contracts/Quest.sol::19 => bool public hasStarted;
quest-protocol/contracts/Quest.sol::20 => bool public isPaused;
quest-protocol/contracts/Quest.sol::21 => string public questId;
quest-protocol/contracts/Quest.sol::22 => uint256 public redeemedTokens;
quest-protocol/contracts/QuestFactory.sol::26 => address public claimSignerAddress;
quest-protocol/contracts/QuestFactory.sol::27 => address public protocolFeeRecipient;
quest-protocol/contracts/QuestFactory.sol::29 => RabbitHoleReceipt public rabbitholeReceiptContract;
quest-protocol/contracts/QuestFactory.sol::31 => uint public questFee;
quest-protocol/contracts/QuestFactory.sol::32 => uint public questIdCount;
quest-protocol/contracts/RabbitHoleReceipt.sol::31 => address public royaltyRecipient;
quest-protocol/contracts/RabbitHoleReceipt.sol::32 => address public minterAddress;
quest-protocol/contracts/RabbitHoleReceipt.sol::33 => uint public royaltyFee;
quest-protocol/contracts/RabbitHoleReceipt.sol::35 => ReceiptRenderer public ReceiptRendererContract;
quest-protocol/contracts/RabbitHoleReceipt.sol::36 => IQuestFactory public QuestFactoryContract;
quest-protocol/contracts/RabbitHoleTickets.sol::22 => address public royaltyRecipient;
quest-protocol/contracts/RabbitHoleTickets.sol::23 => address public minterAddress;
quest-protocol/contracts/RabbitHoleTickets.sol::24 => uint public royaltyFee;
quest-protocol/contracts/RabbitHoleTickets.sol::25 => TicketRenderer public TicketRendererContract
```




### [G02] State variables can be packed into fewer storage slots

#### Impact
If variables occupying the same slot are both written the same 
function or by the constructor avoids a separate Gsset (20000 gas). 
#### Findings:
```
quest-protocol/contracts/QuestFactory.sol::26 => address public claimSignerAddress;
quest-protocol/contracts/QuestFactory.sol::27 => address public protocolFeeRecipient;
quest-protocol/contracts/RabbitHoleReceipt.sol::31 => address public royaltyRecipient;
quest-protocol/contracts/RabbitHoleReceipt.sol::32 => address public minterAddress;
quest-protocol/contracts/RabbitHoleTickets.sol::22 => address public royaltyRecipient;
quest-protocol/contracts/RabbitHoleTickets.sol::23 => address public minterAddress;
```




### [G03] `<array>.length` should not be looked up in every loop of a `for` loop

#### Impact
Even memory arrays incur the overhead of bit tests and bit shifts to 
calculate the array length. Storage array length checks incur an extra 
Gwarmaccess (100 gas) PER-LOOP.
#### Findings:
```
quest-protocol/contracts/Quest.sol::70 => for (uint i = 0; i < tokenIds_.length; i++) {
quest-protocol/contracts/Quest.sol::104 => for (uint i = 0; i < tokens.length; i++) {
```




### [G04] `++i/i++` should be `unchecked{++i}`/`unchecked{++i}` when it is not possible for them to overflow, as is the case when used in `for` and `while` loops


#### Findings:
```
quest-protocol/contracts/Quest.sol::70 => for (uint i = 0; i < tokenIds_.length; i++) {
quest-protocol/contracts/Quest.sol::104 => for (uint i = 0; i < tokens.length; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::117 => for (uint i = 0; i < msgSenderBalance; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::128 => for (uint i = 0; i < msgSenderBalance; i++) {
```




### [G05] It costs more gas to initialize variables to zero than to let the default of zero be applied


#### Findings:
```
quest-protocol/contracts/Quest.sol::70 => for (uint i = 0; i < tokenIds_.length; i++) {
quest-protocol/contracts/Quest.sol::104 => for (uint i = 0; i < tokens.length; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::117 => for (uint i = 0; i < msgSenderBalance; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::128 => for (uint i = 0; i < msgSenderBalance; i++) {
```




### [G06] `++i` costs less gas than `i++`, especially when it’s used in forloops (`--i`/`i--` too)


#### Findings:
```
quest-protocol/contracts/Quest.sol::70 => for (uint i = 0; i < tokenIds_.length; i++) {
quest-protocol/contracts/Quest.sol::104 => for (uint i = 0; i < tokens.length; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::117 => for (uint i = 0; i < msgSenderBalance; i++) {
quest-protocol/contracts/RabbitHoleReceipt.sol::128 => for (uint i = 0; i < msgSenderBalance; i++) {
```



### [G07] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

#### Impact
> When using elements that are smaller than 32 bytes, your 
contract’s gas usage may be higher. This is because the EVM operates on 
32 bytes at a time. Therefore, if the element is smaller than that, the 
EVM must use more operations in order to reduce the size of the element 
from 32 bytes to the desired size.
> 

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)
Use a larger size then downcast where needed
#### Findings:
```
quest-protocol/contracts/ReceiptRenderer.sol::60 => generateAttribute('Reward Address', Strings.toHexString(uint160(rewardAddress_), 20)),
```




### [G08] Expressions for constant values such as a call to `keccak256()`, should use `immutable` rather than `constant`

#### Impact
See [this](https://github.com/ethereum/solidity/issues/9232) issue for a detailed description of the issue
#### Findings:
```
quest-protocol/contracts/QuestFactory.sol::17 => bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
```




### [G09] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function


#### Findings:
```
quest-protocol/contracts/Quest.sol::123 => revert MustImplementInChild();
```




### [G10] `require()` or `revert()` statements that check input arguments should be at the top of the function

#### Impact
Checks that involve constants should come before checks that involve state variables
#### Findings:
```
quest-protocol/contracts/RabbitHoleReceipt.sol::162 => require(QuestFactoryContract != IQuestFactory(address(0)), 'QuestFactory not set');
```




### [G11] Use custom errors rather than `revert()`/`require()` strings to save deployment gas



#### Findings:
```
quest-protocol/contracts/RabbitHoleReceipt.sol::161 => require(_exists(tokenId_), 'ERC721URIStorage: URI query for nonexistent token');
quest-protocol/contracts/RabbitHoleReceipt.sol::162 => require(QuestFactoryContract != IQuestFactory(address(0)), 'QuestFactory not set');
quest-protocol/contracts/RabbitHoleReceipt.sol::182 => require(_exists(tokenId_), 'Nonexistent token');
```




### [G12] Functions guaranteed to revert when called by normal users can be marked `payable`

#### Impact
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
#### Findings:
```
quest-protocol/contracts/Erc1155Quest.sol::54 => function withdrawRemainingTokens(address to_) public override onlyOwner {
quest-protocol/contracts/Erc20Quest.sol::81 => function withdrawRemainingTokens(address to_) public override onlyOwner {
quest-protocol/contracts/Erc20Quest.sol::102 => function withdrawFee() public onlyAdminWithdrawAfterEnd {
quest-protocol/contracts/Quest.sol::50 => function start() public virtual onlyOwner {
quest-protocol/contracts/Quest.sol::57 => function pause() public onlyOwner onlyStarted {
quest-protocol/contracts/Quest.sol::63 => function unPause() public onlyOwner onlyStarted {
quest-protocol/contracts/Quest.sol::96 => function claim() public virtual onlyQuestActive {
quest-protocol/contracts/Quest.sol::150 => function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
quest-protocol/contracts/QuestFactory.sol::142 => function changeCreateQuestRole(address account_, bool canCreateQuest_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::159 => function setClaimSignerAddress(address claimSignerAddress_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::165 => function setProtocolFeeRecipient(address protocolFeeRecipient_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::172 => function setRabbitHoleReceiptContract(address rabbitholeReceiptContract_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::179 => function setRewardAllowlistAddress(address rewardAddress_, bool allowed_) public onlyOwner {
quest-protocol/contracts/QuestFactory.sol::186 => function setQuestFee(uint256 questFee_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::65 => function setReceiptRenderer(address receiptRenderer_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::71 => function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::77 => function setQuestFactory(address questFactory_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::83 => function setMinterAddress(address minterAddress_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::90 => function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
quest-protocol/contracts/RabbitHoleReceipt.sol::98 => function mint(address to_, string memory questId_) public onlyMinter {
quest-protocol/contracts/RabbitHoleTickets.sol::54 => function setTicketRenderer(address ticketRenderer_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::60 => function setRoyaltyRecipient(address royaltyRecipient_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::66 => function setRoyaltyFee(uint256 royaltyFee_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::73 => function setMinterAddress(address minterAddress_) public onlyOwner {
quest-protocol/contracts/RabbitHoleTickets.sol::83 => function mint(address to_, uint256 id_, uint256 amount_, bytes memory data_) public onlyMinter {
```


### [G13] Use `calldata` instead of `memory` for function parameters

#### Impact
Use calldata instead of memory for function parameters. Having function arguments use calldata instead of memory can save gas.
#### Findings:
```
quest-protocol/contracts/QuestFactory.sol::193 => function getNumberMinted(string memory questId_) external view returns (uint) {
quest-protocol/contracts/QuestFactory.sol::199 => function questInfo(string memory questId_) external view returns (address, uint, uint) {
quest-protocol/contracts/QuestFactory.sol::210 => function recoverSigner(bytes32 hash_, bytes memory signature_) public pure returns (address) {
quest-protocol/contracts/QuestFactory.sol::219 => function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
quest-protocol/contracts/RabbitHoleReceipt.sol::98 => function mint(address to_, string memory questId_) public onlyMinter {
quest-protocol/contracts/RabbitHoleReceipt.sol::110 => string memory questId_,
quest-protocol/contracts/RabbitHoleReceipt.sol::164 => string memory questId = questIdForTokenId[tokenId_];
quest-protocol/contracts/ReceiptRenderer.sol::82 => function generateAttribute(string memory key, string memory value) public pure returns (string memory) {
quest-protocol/contracts/ReceiptRenderer.sol::100 => function generateSVG(uint tokenId_, string memory questId_) public pure returns (string memory) {
quest-protocol/contracts/interfaces/IQuestFactory.sol::19 => function questInfo(string memory questId_) external view returns (address, uint, uint);
```



### [G14] Multiple `address` mappings can be combined into a single `mapping` of an `address` to a `struct`, where appropriate

#### Impact
Saves a storage slot for the mapping. Depending on the circumstances 
and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. 
Reads and subsequent writes can also be cheaper when a function requires
 both values and they both fit in the same storage slot
#### Findings:
```
quest-protocol/contracts/QuestFactory.sol::20 => mapping(address => bool) addressMinted;
quest-protocol/contracts/QuestFactory.sol::30 => mapping(address => bool) public rewardAllowlist;
```



### [G15] Using `bools` for storage incurs overhead


#### Findings:
```
quest-protocol/contracts/Quest.sol::19 => bool public hasStarted;
quest-protocol/contracts/Quest.sol::20 => bool public isPaused;
quest-protocol/contracts/Quest.sol::24 => mapping(uint256 => bool) private claimedList;
quest-protocol/contracts/QuestFactory.sol::20 => mapping(address => bool) addressMinted;
quest-protocol/contracts/QuestFactory.sol::30 => mapping(address => bool) public rewardAllowlist;
```


### [G16] Using `private` rather than `public` for constants, saves gas

#### Impact
If needed, the value can be read from the verified contract source 
code. 
#### Findings:
```
quest-protocol/contracts/QuestFactory.sol::17 => bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
```






### [G17] Empty blocks should be removed or emit something

#### Impact
The code should be refactored such that they no longer exist, or the 
block should do something useful, such as emitting an event or 
reverting. If the contract is meant to be extended, the contract should 
be abstract and the function signatures be added without 
any default implementation. If the block is an empty if-statement block 
to avoid doing subsequent checks in the else-if/else conditions, the 
else-if/else conditions should be nested under the negation of the 
if-statement, because they involve different classes of checks, which 
may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})
#### Findings:
```
quest-protocol/contracts/Erc1155Quest.sol::30 => ){}
quest-protocol/contracts/Quest.sol::150 => function withdrawRemainingTokens(address to_) public virtual onlyOwner onlyAdminWithdrawAfterEnd {}
quest-protocol/contracts/QuestFactory.sol::35 => constructor() initializer {}
```





