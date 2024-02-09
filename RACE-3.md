**Note**: All 8 questions in this quiz are based on the _InSecureumNFT_ contract snippet. This is the same contract snippet you will see for all the 8 questions in this quiz.

_InSecureumNFT_ is a NFT project that aims to distribute CryptoSAFU NFTs to its community where most of them are fairdropped based on past contributions and a few are sold. CryptoSAFUs with lower IDs have more unique traits, may be valued higher and therefore require a random distribution for fairness. Assume that all strictly required ERC721 functionality (not shown) and any other required functionality (not shown) are implemented correctly. Only functionality specific to the sale and minting of NFTs is shown in this contract snippet.
```solidity
pragma solidity 0.8.0;

interface ERC721TokenReceiver{function onERC721Received(address _operator, address _from, uint256 _tokenId, bytes calldata _data) external returns(bytes4);}

// Assume that all strictly required ERC721 functionality (not shown) is implemented correctly
// Assume that any other required functionality (not shown) is implemented correctly
contract InSecureumNFT {
    bytes4 internal constant MAGIC_ERC721_RECEIVED = 0x150b7a02;
    uint public constant TOKEN_LIMIT = 10; // 10 for testing, 13337 for production
    uint public constant SALE_LIMIT = 5; // 5 for testing, 1337 for production

    mapping (uint256 => address) internal idToOwner;
    uint internal numTokens = 0;
    uint internal numSales = 0;
    address payable internal deployer;
    address payable internal beneficiary;
    bool public publicSale = false;
    uint private price;
    uint public saleStartTime;
    uint public constant saleDuration = 13*13337; // 13337 blocks assuming 13s block times 
    uint internal nonce = 0;
    uint[TOKEN_LIMIT] internal indices;
 
    constructor(address payable _beneficiary) {
        deployer = payable(msg.sender);
        beneficiary = _beneficiary;
    }

    function startSale(uint _price) external {
        require(msg.sender == deployer || _price != 0, "Only deployer and price cannot be zero");
        price = _price;
        saleStartTime = block.timestamp;
        publicSale = true;
    }

    function isContract(address _addr) internal view returns (bool addressCheck) {
        uint256 size;
        assembly { size := extcodesize(_addr) }
        addressCheck = size > 0;
    }

    function randomIndex() internal returns (uint) {
        uint totalSize = TOKEN_LIMIT - numTokens;
        uint index = uint(keccak256(abi.encodePacked(nonce, msg.sender, block.difficulty, block.timestamp))) % totalSize;
        uint value = 0;
        if (indices[index] != 0) {
            value = indices[index];
        } else {
            value = index;
        }
        if (indices[totalSize - 1] == 0) {
            indices[index] = totalSize - 1;
        } else {
            indices[index] = indices[totalSize - 1];
        }
        nonce += 1;
        return (value + 1);
    }

    // Calculate the mint price
    function getPrice() public view returns (uint) {
        require(publicSale, "Sale not started.");
        uint elapsed = block.timestamp - saleStartTime;
        if (elapsed > saleDuration) {
            return 0;
        } else {
            return ((saleDuration - elapsed) * price) / saleDuration;
        }
    }
    
    // SALE_LIMIT is 1337 
    // Rest i.e. (TOKEN_LIMIT - SALE_LIMIT) are reserved for community distribution (not shown)
    function mint() external payable returns (uint) {
        require(publicSale, "Sale not started.");
        require(numSales < SALE_LIMIT, "Sale limit reached.");
        numSales++;
        uint salePrice = getPrice();
        require((address(this)).balance >= salePrice, "Insufficient funds to purchase.");
        if ((address(this)).balance >= salePrice) {
            payable(msg.sender).transfer((address(this)).balance - salePrice);
        }
        return _mint(msg.sender);
    }

    // TOKEN_LIMIT is 13337
    function _mint(address _to) internal returns (uint) {
        require(numTokens < TOKEN_LIMIT, "Token limit reached.");
        // Lower indexed/numbered NFTs have rare traits and may be considered
        // as more valuable by buyers => Therefore randomize
        uint id = randomIndex();
        if (isContract(_to)) {
            bytes4 retval = ERC721TokenReceiver(_to).onERC721Received(msg.sender, address(0), id, "");
            require(retval == MAGIC_ERC721_RECEIVED);
        }
        require(idToOwner[id] == address(0), "Cannot add, already owned.");
        idToOwner[id] = _to;
        numTokens = numTokens + 1;
        beneficiary.transfer((address(this)).balance);
        return id;
    }
}
```

---

**[Q1] Missing zero-address check(s) in the contract**  

(A): May allow anyone to start the sale  
(B): May put the NFT sale proceeds at risk  
(C): May burn the newly minted NFTs  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
B
</b></details>

---

**[Q2] Given that lower indexed/numbered CryptoSAFU NFTs have rarer traits (and are considered more valuable as commented in `_mint`), the implementation of _InSecureumNFT_ is susceptible to the following exploits**  

(A): Buyers can repeatedly mint and revert until they receive desired NFT  
(B): Buyers can generate addresses to mint until they receive desired NFT  
(C): Miners can manipulate _block.timestamp_ to facilitate minting of desired NFT  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
A,B,C
</b></details>

---

**[Q3] The _getPrice()_ function**  

(A): Is expected to reduce the mint price over time after sale starts  
(B): Allows free mints after ~13337 blocks from when _startSale()_ is called  
(C): Visibility should be changed to _external_  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
A,B
</b></details>  

---

**[Q4] _InSecureumNFT_ contract is**  

(A): Not susceptible to reentrancy given the absence of external contract calls  
(B): Not susceptible to integer overflow/wrapping given the compiler version used and the absence of unchecked blocks  
(C): Susceptible to reentrancy during minting  
(D): Perfectly safe for production  

<details><summary><b>[Answers]</b></summary><b>
B,C
</b></details>

---

**[Q5] Assuming _InSecureumNFT_ contract is deployed in production (i.e. live for users) on mainnet without any changes to shown code**

(A): Use of evident test configuration will cause fewer NFTs to be minted than expected in production  
(B): Illustrates the lack of best-practice for test parameterization to be removed or kept separate from production code  
(C): It will behave as documented in code to mint the expected number of NFTs in production  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
A,B
</b></details>

---

**[Q6] The function _startSale()_**  

(A): May be successfully called/executed by anyone  
(B): May be successfully called/executed with `_price` of 0  
(C): Must be called for minting to happen successfully  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
A,B,C
</b></details>  

---

**[Q7] The minting of NFTs**  

(A): Requires an exact amount of ETH to be paid by the buyer  
(B): Refunds excess ETH paid by buyer back to the buyer  
(C): Transfers the NFT _salePrice_ to the _beneficiary_ address  
(D): May be optimized to prevent any zero ETH transfers in its refund mechanism  

<details><summary><b>[Answers]</b></summary><b>
B,C,D
</b></details>  

---

**[Q8] The NFT sale**  

(A): May be restarted by anyone any number of times  
(B): Can be started exactly once by _deployer_  
(C): Is missing an additional check on _publicSale_  
(D): Is missing an event emit in _startSale_  

<details><summary><b>[Answers]</b></summary><b>
A,C,D
</b></details>

---
