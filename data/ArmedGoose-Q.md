SVG is rendering with unsanitized special charaters.
The rendering function is using user input (string) (string memory questId_) to output an SVG. This might be lead to issues in the future, since SVG format allows for embedding javascript scripts inside, so potentially, if that function has any other use in the future, it might be abused to generate malicious images.
However, at the moment probability is low, since it uses existing questId is created by administrative role, and anyone in posession of administrative role have plenty of other attack vectors than xssing users. 

Occurence:
https://github.com/rabbitholegg/quest-protocol/blob/main/contracts/ReceiptRenderer.sol#L100

Remediation:
Filter special characters, at least <, >, ", ' ideally allow only alphanumeric ones in the function.

References:
https://github.com/code-423n4/2022-01-timeswap-findings/issues/131
https://securiumsolutions.com/blog/xss-through-svg-file-upload/