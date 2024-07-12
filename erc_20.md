# Smart Contract Function Analysis

## Introduction

- **Protocol Name:** Centrifuge (CFG)

- **Category:** DeFi
- **Smart Contract:** ERC20 Token Contract

## Function Analysis

### Function Name: `permit`

- **Block Explorer Link:** https://etherscan.io/token/0xc221b7e65ffc80de234bbb6667abdd46593d34f0#code

### Function Code:

```solidity
function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external {
    require(deadline >= block.timestamp, 'cent/past-deadline');
    bytes32 digest = keccak256(
        abi.encodePacked(
            '\x19\x01',
            DOMAIN_SEPARATOR,
            keccak256(abi.encode(PERMIT_TYPEHASH, owner, spender, value, nonces[owner]++, deadline))
        )
    );
    address recoveredAddress = ecrecover(digest, v, r, s);
    require(recoveredAddress != address(0) && recoveredAddress == owner, 'cent-erc20/invalid-sig');
    allowance[owner][spender] = value;
    emit Approval(owner, spender, value);
}
```




### Used Encoding/Decoding or Call Method: `abi.encode`, `abi.encodePacked`

### Explanation

#### Purpose

The `permit` function allows an address to approve another address to spend tokens on its behalf without requiring an on-chain transaction from the owner. This is accomplished by validating an off-chain signature.

### Detailed Usage

#### Deadline Check:
The function ensures that the provided deadline has not yet passed, preventing expired approvals.

#### Creating the Message Digest:
The function constructs a digest using `abi.encodePacked` and `abi.encode`:
- `abi.encodePacked('\x19\x01', DOMAIN_SEPARATOR, ...)` creates a packed encoding of the prefix and the domain separator.
- `abi.encode(PERMIT_TYPEHASH, owner, spender, value, nonces[owner]++, deadline)` encodes the permit type hash and relevant parameters into a byte array.
- The combined packed and encoded data is then hashed using `keccak256`.

#### Signature Verification:
The function recovers the signer's address from the provided signature using `ecrecover`.
- It checks that the recovered address matches the owner and is not zero.

#### Setting Allowance:
If the signature is valid, the function updates the allowance mapping to grant `spender` permission to spend `value` tokens on behalf of `owner`.

#### Event Emission:
The function emits an `Approval` event to reflect the approval change on the blockchain.

### Impact

#### Functionality:
The `permit` function facilitates gasless token approvals, enhancing the user experience by allowing users to approve token transfers without spending ETH for transaction fees.
- It streamlines interactions with DeFi protocols by enabling off-chain approvals, reducing the need for multiple on-chain transactions.

#### Security:
By using `abi.encode` and `abi.encodePacked`, the function ensures that the message digest is securely and uniquely constructed.
- The use of `ecrecover` to validate the signature guarantees that only the legitimate owner can authorize approvals.

### Conclusion
The `permit` function in the Cent protocol's ERC20 token contract leverages `abi.encode` and `abi.encodePacked` to create a secure and efficient method for off-chain token approvals. This function enhances the flexibility and usability of the DeFi protocol by allowing gasless approvals while ensuring the integrity and security of the approval process.
