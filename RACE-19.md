**Note**: All 8 questions in this RACE are based on the below contract. This is the same contract you will see for all the 8 questions in this RACE. The question is below the shown contract.
```
pragma solidity 0.8.20;

import {ERC721Burnable} from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
import {ERC1155Burnable} from "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155Burnable.sol";
import {Address} from "@openzeppelin/contracts/utils/Address.sol";


contract WalletFactory {
   using Address for address;
   address immutable implementation;

   constructor(address _implementation) {
       implementation = _implementation;
   }

   function deployAndLoad(uint256 salt) external payable returns (address addr) {
       addr = deploy(salt);
       payable(addr).send(msg.value);
   }

   function deploy(uint256 salt) public returns (address addr) {
       bytes memory code = implementation.code;
       assembly {
           addr := create2(0, add(code, 0x20), mload(code), salt)
       }
    }
}  


contract Wallet {

   struct Transaction {
       address from;
       address to;
       uint256 value;
       bytes data;
   }

   uint256 nonce;

   receive() external payable {}
   fallback() external payable {}

   function execute(Transaction calldata transaction, bytes calldata signature) public payable {
       bytes32 hash = keccak256(abi.encode(address(this), nonce, transaction));
       bytes32 r = readBytes32(signature, 0);
       bytes32 s = readBytes32(signature, 32);
       uint8 v = uint8(signature[64]);
       address signer = ecrecover(hash, v, r, s);

       if (signer == msg.sender || signer == transaction.from) {
           address to = transaction.to;
           uint256 value = transaction.value;
           bytes memory data = transaction.data;

           assembly {
               let res := call(gas(), to, value, add(data, 0x20), mload(data), 0, 0)
           }
           return;
       }

       nonce++;
   }

   function executeMultiple(Transaction[] calldata transactions, bytes[] calldata signatures) external payable {
       for(uint256 i = 0; i < transactions.length; ++i) execute(transactions[i], signatures[i]);
   }

   function readBytes32(bytes memory b, uint256 index) internal pure returns (bytes32 result) {
       index += 32;
       require(b.length >= index);

       assembly {
           result := mload(add(b, index))
       }
   }

   function burnNFT(address owner, ERC721Burnable nftContract, uint256 id) external {
       require(msg.sender == owner, "Unauthorized");
       nftContract.burn(id);
   }

   function burnERC1155(ERC1155Burnable semiFungibleToken, uint256 id, uint256 amount) external {
       semiFungibleToken.burn(msg.sender, id, amount);
   }
}
```
---
**[Q1] The deployment concern(s) here for different EVM-compatible chains is/are:** \
(A): `receive` method behavior might be undefined \
(B): The presence of `ecrecover` precompile is potentially dangerous \
(C): Not all opcodes in the bytecode are guaranteed to be supported \
(D): None of the above 

**[Answers]: C**

---
**[Q2] The security concern(s) in `WalletFactory` is/are:** \
(A): ETH funds may get stuck in it forever \
(B): The `deploy` method is not marked as `payable` \
(C): No access control on wallet deployment \
(D): Deployment may silently fail 

**[Answers]: A, D**

---
**[Q3] Design flaw(s) of `Wallet` is/are:** \
(A): Missing wallet owner role and appropriate access control \
(B): Inability to rescue stuck tokens \
(C): Assembly usage is unsafe for the Yul IR pipeline \
(D): Calling a `payable` method in a for-loop

**[Answers]: A**

---
**[Q4] The security concern(s) with hashing of `transaction` parameter in `execute` is/are:** \
(A): Cross-contract replay attacks \
(B): Cross-chain replay attacks \
(C): `keccak256` hash collision attacks \
(D): Reentrancy attacks

**[Answers]: B**

---
**[Q5] If the hashed payload in `execute` were to exclude a nonce, the security concern(s) with `ecrecover` would be:** \
(A): Signature malleability by flipping the “s” or “v” values \
(B): Signature malleability by using compact signatures \
(C): Signature malleability by hash collisions \
(D): Forcefully reverting transactions

**[Answers]: A**

---
**[Q6] The security concern(s) with `Wallet` is/are:** \
(A): Ether sent to the contract will be stuck forever \
(B): Anyone can execute arbitrary calls \
(C): Anyone can steal the contract ETH balance \
(D): None of the above

**[Answers]: B, C**

---
**[Q7] The nonce best practice(s) _not_ followed correctly is/are:** \
(A): Nonce is not incremented before the low-level call \
(B): Nonce is not guaranteed to be included in the signature \
(C): Nonce is not incremented correctly on transaction execution \
(D): None of the above

**[Answers]: A, C**

---
**[Q8] The security concern(s) with `Wallet` contract related to ERC721 tokens is/are:** \
(A): There is no way to get ERC721 tokens out of the contract \
(B): Failure to receive ERC721 tokens depending on the transfer method \
(C): Failure to receive any ERC721 tokens \
(D): Unauthorized burning of ERC721 tokens

**[Answers]: B, D**

---
