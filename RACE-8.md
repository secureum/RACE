**Note**: All 8 questions in this RACE are based on the InSecureumERC721 contract. This is the same contract you will see for all the 8 questions in this RACE. InSecureumERC721 is adapted from a well-known contract. The question is below the shown contract.

```
pragma solidity >=0.8.0;

abstract contract InSecureumERC721 {

   event Transfer(address indexed from, address indexed to, uint256 indexed id);

   event Approval(address indexed owner, address indexed spender, uint256 indexed id);

   event ApprovalForAll(address indexed owner, address indexed operator, bool approved);

   string public name;

   string public symbol;

   function tokenURI(uint256 id) public view virtual returns (string memory);

   mapping(uint256 => address) internal _ownerOf;

   mapping(address => uint256) internal _balanceOf;

   function ownerOf(uint256 id) public view virtual returns (address owner) {
       require((owner = _ownerOf[id]) != address(0), "NOT_MINTED");
   }

   function balanceOf(address owner) public view virtual returns (uint256) {
       require(owner != address(0), "ZERO_ADDRESS");

       return _balanceOf[owner];
   }

   mapping(uint256 => address) public getApproved;

   mapping(address => mapping(address => bool)) public isApprovedForAll;

   constructor(string memory _name, string memory _symbol) {
       name = _name;
       symbol = _symbol;
   }

   function approve(address spender, uint256 id) public virtual {
       address owner = _ownerOf[id];

       require(msg.sender == owner || isApprovedForAll[owner][msg.sender], "NOT_AUTHORIZED");

       getApproved[id] = spender;

       emit Approval(owner, spender, id);
   }

   function setApprovalForAll(address operator, bool approved) public virtual {
       isApprovedForAll[msg.sender][operator] = approved;

       emit ApprovalForAll(msg.sender, operator, approved);
   }

   function transferFrom(
       address from,
       address to,
       uint256 id
   ) public virtual {
       require(to != address(0), "INVALID_RECIPIENT");

       require(
           msg.sender == from || isApprovedForAll[from][msg.sender] || msg.sender == getApproved[id],
           "NOT_AUTHORIZED"
       );

       unchecked {
           _balanceOf[from]--;

           _balanceOf[to]++;
       }

       _ownerOf[id] = to;

       delete getApproved[id];

       emit Transfer(from, to, id);
   }

   function safeTransferFrom(
       address from,
       address to,
       uint256 id
   ) public virtual {
       transferFrom(from, to, id);

       require(
           to.code.length == 0 ||
               ERC721TokenReceiver(to).onERC721Received(msg.sender, from, id, "") ==
               ERC721TokenReceiver.onERC721Received.selector,
           "UNSAFE_RECIPIENT"
       );
   }

   function safeTransferFrom(
       address from,
       address to,
       uint256 id,
       bytes calldata data
   ) public virtual {
       transferFrom(from, to, id);

       require(
           to.code.length == 0 ||
               ERC721TokenReceiver(to).onERC721Received(msg.sender, from, id, data) ==
               ERC721TokenReceiver.onERC721Received.selector,
           "UNSAFE_RECIPIENT"
       );
   }

   function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {
       return
           interfaceId == 0x01ffc9a7 || // ERC165 Interface ID for ERC165
           interfaceId == 0x80ac58cd || // ERC165 Interface ID for ERC721
           interfaceId == 0x5b5e139f; // ERC165 Interface ID for ERC721Metadata
   }

   function _mint(address to, uint256 id) internal virtual {
       require(to != address(0), "INVALID_RECIPIENT");

       require(_ownerOf[id] == address(0), "ALREADY_MINTED");

       unchecked {
           _balanceOf[to]++;
       }

       _ownerOf[id] = to;

       emit Transfer(address(0), to, id);
   }

   function _burn(uint256 id) external virtual {
       address owner = _ownerOf[id];

       require(owner != address(0), "NOT_MINTED");

       unchecked {
           _balanceOf[owner]--;
       }

       delete _ownerOf[id];

       delete getApproved[id];

       emit Transfer(owner, address(0), id);
   }

   function _safeMint(address to, uint256 id) internal virtual {
       _mint(to, id);

       require(
           to.code.length == 0 ||
               ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, "") ==
               ERC721TokenReceiver.onERC721Received.selector,
           "UNSAFE_RECIPIENT"
       );
   }

   function _safeMint(
       address to,
       uint256 id,
       bytes memory data
   ) internal virtual {
       _mint(to, id);

       require(
           to.code.length == 0 ||
               ERC721TokenReceiver(to).onERC721Received(msg.sender, address(0), id, data) ==
               ERC721TokenReceiver.onERC721Received.selector,
           "UNSAFE_RECIPIENT"
       );
   }
}

abstract contract ERC721TokenReceiver {
   function onERC721Received(
       address,
       address,
       uint256,
       bytes calldata
   ) external virtual returns (bytes4) {
       return ERC721TokenReceiver.onERC721Received.selector;
   }
}
```

---

**[Q1] Which of the following is/are true?**

(A): NFT ownership is tracked by `_ownerOf`  
(B): NFT balance is tracked by `_balanceOf`  
(C): NFT approvals are tracked by `getApproved`  
(D): NFT operator can transfer all of owner’s NFTs  

**[Answers]: A, B, C, D**

---

**[Q2] _InSecureumERC721_ recognizes the following role(s)**

(A): Owner  
(B): Spender (Approved address)  
(C): Operator  
(D): None of the above  

**[Answers]: A, B, C**

---

**[Q3] The security concern(s) addressed explicitly in `_mint` include**

(A): Prevent minting to zero address  
(B): Prevent reminting of NFTs  
(C): Transparency by emitting event  
(D): None of the above  

**[Answers]: A, B, C**

---

**[Q4] The security concerns in `_burn` include**

(A): Anyone can arbitrarily burn NFTs  
(B): Potential integer underflow because of unchecked  
(C): Incorrect emission of event  
(D): None of the above  

**[Answers]: A**

---

**[Q5] The security concern(s) addressed explicitly in `_safeMint` include**

(A): Validating if the recipient is an EOA  
(B): Ensuring that the recipient can only be an EOA  
(C): Validating if the recipient is an ERC721 aware contract  
(D): None of the above  

**[Answers]: A, C**

---

**[Q6] Function approve**

(A): Allows the NFT owner to approve a spender  
(B): Allows the NFT spender to approve an operator  
(C): Allows the NFT operator to approve a spender  
(D): None of the above  

**[Answers]: A, C**

---

**[Q7] Function `setApprovalForAll`**

(A): Approves `msg.sender` to manage operator’s NFTs  
(B): Gives everyone approval to manage `msg.sender`’s NFTs  
(C): Revokes everyone’s approvals to manage `msg.sender`’s NFTs  
(D): None of the above  

**[Answers]: D**

---

**[Q8] The security concern(s) in `transferFrom` include**

(A): Allowing the `msg.sender` to transfer any NFT  
(B): NFTs potentially stuck in recipient contracts  
(C): Potential integer underflow  
(D): None of the above  

**[Answers]: A, B, C**

---
