**Note**: All 8 questions in this RACE are based on the _InSecureumApe_ contract. This is the same contract you will see for all the 8 questions in this RACE. _InSecureumApe_ is adapted from a well-known contract. The question is below the shown contract.

```solidity
pragma solidity ^0.7.0;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v3.4/contracts/access/Ownable.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v3.4/contracts/token/ERC721/ERC721.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v3.4/contracts/math/SafeMath.sol";


contract InSecureumApe is ERC721, Ownable {

    using SafeMath for uint256;

    string public IA_PROVENANCE = "";
    uint256 public startingIndexBlock;
    uint256 public startingIndex;
    uint256 public constant apePrice = 800000000000000000; //0.08 ETH
    uint public constant maxApePurchase = 20;
    uint256 public MAX_APES;
    bool public saleIsActive = false;
    uint256 public REVEAL_TIMESTAMP;

    constructor(string memory name, string memory symbol, uint256 maxNftSupply, uint256 saleStart) ERC721(name, symbol) {
        MAX_APES = maxNftSupply;
        REVEAL_TIMESTAMP = saleStart + (86400 * 9);
    }

    function withdraw() public onlyOwner {
        uint balance = address(this).balance;
        msg.sender.transfer(balance);
    }

    function reserveApes() public onlyOwner {        
        uint supply = totalSupply();
        uint i;
        for (i = 0; i < 30; i++) {
            _safeMint(msg.sender, supply + i);
        }
    }

    function setRevealTimestamp(uint256 revealTimeStamp) public onlyOwner {
        REVEAL_TIMESTAMP = revealTimeStamp;
    } 

    function setProvenanceHash(string memory provenanceHash) public onlyOwner {
        IA_PROVENANCE = provenanceHash;
    }

    function setBaseURI(string memory baseURI) public onlyOwner {
        _setBaseURI(baseURI);
    }

    function flipSaleState() public onlyOwner {
        saleIsActive = !saleIsActive;
    }

    function mintApe(uint numberOfTokens) public payable {
        require(saleIsActive, "Sale must be active to mint Ape");
        require(numberOfTokens < maxApePurchase, "Can only mint 20 tokens at a time");
        require(totalSupply().add(numberOfTokens) <= MAX_APES, "Purchase would exceed max supply of Apes");
        require(apePrice.mul(numberOfTokens) <= msg.value, "Ether value sent is not correct");

        for(uint i = 0; i < numberOfTokens; i++) {
            uint mintIndex = totalSupply();
            if (totalSupply() < MAX_APES) {
                _safeMint(msg.sender, mintIndex);
            }
        }

        // If we haven't set the starting index and this is either 1) the last saleable token or 2) the first token to be sold after
        // the end of pre-sale, set the starting index block
        if (startingIndexBlock == 0 && (totalSupply() == MAX_APES || block.timestamp >= REVEAL_TIMESTAMP)) {
            startingIndexBlock = block.number;
        } 
    }

    function setStartingIndex() public {
        require(startingIndex == 0, "Starting index is already set");
        require(startingIndexBlock != 0, "Starting index block must be set");

        startingIndex = uint(blockhash(startingIndexBlock)) % MAX_APES;
        if (block.number.sub(startingIndexBlock) > 255) {
            startingIndex = uint(blockhash(block.number - 1)) % MAX_APES;
        }
        if (startingIndex == 0) {
            startingIndex = startingIndex.add(1);
        }
    }

    function emergencySetStartingIndexBlock() public onlyOwner {
        require(startingIndex == 0, "Starting index is already set");
        startingIndexBlock = block.number;
    }
}
```

---

**[Q1] The mint price of an _InSecureumApe_ is**

(A): 0.0008 ETH  
(B): 0.008 ETH  
(C): 0.08 ETH  
(D): 0.8 ETH  

<details><summary><b>[Answers]</b></summary><b>
D
</b></details>

---

**[Q2] The security concern(s) with _InSecureumApe_ access control is/are**

(A): Owner can arbitrarily pause public minting of _InSecureumApe_  
(B): Owner can arbitrarily mint _InSecureumApe_  
(C): Single-step ownership change  
(D): Missing event emits in and time-delayed effects of owner functions  

<details><summary><b>[Answers]</b></summary><b>
A, B, C, D
</b></details>

---

**[Q3] The security concern(s) with _InSecureumApe_ constructor is/are**

(A): Missing sanity/threshold check on `maxNftSupply`  
(B): Missing sanity/threshold check on `saleStart`  
(C): Potential integer overflow  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
A, B, C
</b></details>

---

**[Q4] The total number of _InSecureumApe_ that can ever be minted is**

(A): `maxApePurchase`  
(B): `MAX_APES`  
(C): `MAX_APES` + 30  
(D): `type(uint256).max`  

<details><summary><b>[Answers]</b></summary><b>
D
</b></details>

---

**[Q5] The public minting of _InSecureumApe_**

(A): Must be paid the exact amount in Ether  
(B): May be performed 19 NFTs at a time  
(C): Uses `_safeMint` to prevent locked/stuck NFTs  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
B, C
</b></details>

---

**[Q6] The security concern(s) with _InSecureumApe_ is/are**

(A): Use of a floating pragma and an older compiler version  
(B): Oracle price manipulation  
(C): Reentrancy allowing bypass of `maxApePurchase` check  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
A, C
</b></details>

---

**[Q7] The starting index determination**

(A): Is meant to randomize NFT reveal post-mint  
(B): Can be triggered by the owner at any time  
(C): May be triggered only 9 days after sale start  
(D): Accounts for the fact that EVM only stores previous 256 block hashes  

<details><summary><b>[Answers]</b></summary><b>
A, B, D
</b></details>

---

**[Q8] Potential gas optimization(s) in _InSecureumApe_ is/are**

(A): Caching of storage variables  
(B): Avoiding initializations of variables to default values of their types  
(C): Use of immutables  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
A, B, C
</b></details>

---
