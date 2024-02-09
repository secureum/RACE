**Note**: All 8 questions in this quiz are based on the _InSecureum_ contract. This is the same contract you will see for all the 8 questions in this quiz. _InSecureum_ is adapted from a well-known contract. The question is below the shown contract.

```solidity
pragma solidity ^0.8.0;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC1155/IERC1155.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC1155/IERC1155Receiver.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC1155/extensions/IERC1155MetadataURI.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Context.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/introspection/ERC165.sol";


contract InSecureum is Context, ERC165, IERC1155, IERC1155MetadataURI {

    mapping(uint256 => mapping(address => uint256)) private _balances;
    mapping(address => mapping(address => bool)) private _operatorApprovals;
    string private _uri;

    constructor(string memory uri_) {
        _setURI(uri_);
    }

    function supportsInterface(bytes4 interfaceId) public view virtual override(ERC165, IERC165) returns (bool) {
        return
            interfaceId == type(IERC1155).interfaceId ||
            interfaceId == type(IERC1155MetadataURI).interfaceId ||
            super.supportsInterface(interfaceId);
    }

    function uri(uint256) public view virtual override returns (string memory) {
        return _uri;
    }

    function balanceOf(address account, uint256 id) public view virtual override returns (uint256) {
        require(account != address(0), "ERC1155: balance query for the zero address");
        return _balances[id][account];
    }

    function balanceOfBatch(address[] memory accounts, uint256[] memory ids)
        public view virtual override returns (uint256[] memory) {
        uint256[] memory batchBalances = new uint256[](accounts.length);
        for (uint256 i = 0; i < accounts.length; ++i) {
            batchBalances[i] = balanceOf(accounts[i], ids[i]);
        }
        return batchBalances;
    }

    function setApprovalForAll(address operator, bool approved) public virtual override {
        _setApprovalForAll(_msgSender(), operator, approved);
    }

    function isApprovedForAll(address account, address operator) public view virtual override returns (bool) {
        return _operatorApprovals[account][operator];
    }

    function safeTransferFrom(
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes memory data
    ) public virtual override {
        require(
            from == _msgSender() || isApprovedForAll(from, _msgSender()),
            "ERC1155: caller is not owner nor approved"
        );
        _safeTransferFrom(from, to, id, amount, data);
    }

    function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts,
        bytes memory data
    ) public virtual override {
        require(
            from == _msgSender() || isApprovedForAll(from, _msgSender()),
            "ERC1155: transfer caller is not owner nor approved"
        );
        _safeBatchTransferFrom(from, to, ids, amounts, data);
    }

    function _safeTransferFrom(
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes memory data
    ) public virtual {
        address operator = _msgSender();
        uint256 fromBalance = _balances[id][from];
        unchecked {
             fromBalance = fromBalance - amount;
        }
        _balances[id][from] = fromBalance;
        _balances[id][to] += amount;
        emit TransferSingle(operator, from, to, id, amount);
        _doSafeTransferAcceptanceCheck(operator, from, to, id, amount, data);
    }
    function _safeBatchTransferFrom(
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts,
        bytes memory data
    ) internal virtual {
        require(to != address(0), "ERC1155: transfer to the zero address");
        address operator = _msgSender();
        for (uint256 i = 0; i < ids.length; ++i) {
            uint256 id = ids[i];
            uint256 amount = amounts[i];
            uint256 fromBalance = _balances[id][from];
            fromBalance = fromBalance - amount;
            _balances[id][to] += amount;
        }
        emit TransferBatch(operator, from, to, ids, amounts);
        _doSafeBatchTransferAcceptanceCheck(operator, from, to, ids, amounts, data);
    }

    function _setURI(string memory newuri) internal virtual {
        _uri = newuri;
    }

    function _mint(
        address to,
        uint256 id,
        uint256 amount,
        bytes memory data
    ) internal virtual {
        require(to != address(0), "ERC1155: mint to the zero address");
        address operator = _msgSender();
        _balances[id][to] += amount;
        emit TransferSingle(operator, address(0), to, id, amount);
        _doSafeTransferAcceptanceCheck(operator, address(0), to, id, amount, data);
    }

    function _mintBatch(
        address to,
        uint256[] memory ids,
        uint256[] memory amounts,
        bytes memory data
    ) internal virtual {
        address operator = _msgSender();
        require(operator != address(0), "ERC1155: mint from the zero address");
        for (uint256 i = 0; i < ids.length; i++) {
            _balances[ids[i]][to] += amounts[i];
        }
        emit TransferBatch(operator, address(0), to, amounts, ids);
        _doSafeBatchTransferAcceptanceCheck(operator, address(0), to, ids, amounts, data);
    }

    function _burn(
        address from,
        uint256 id,
        uint256 amount
    ) internal virtual {
        require(from != address(0), "ERC1155: burn from the zero address");
        address operator = _msgSender();
        uint256 fromBalance = _balances[id][from];
        _balances[id][from] = fromBalance - amount;
        emit TransferSingle(operator, from, address(0), id, amount);
    }

    function _burnBatch(
        address from,
        uint256[] memory ids,
        uint256[] memory amounts
    ) internal virtual {
        require(from != address(0), "ERC1155: burn from the zero address");
        address operator = _msgSender();
        for (uint256 i = 0; i < ids.length; i++) {
            uint256 id = ids[i];
            uint256 amount = amounts[i];
            uint256 fromBalance = _balances[id][from];
            require(fromBalance >= amount, "ERC1155: burn amount exceeds balance");
            unchecked {
                _balances[id][from] = fromBalance - amount;
            }
        }
        emit TransferBatch(operator, from, address(0), ids, amounts);
    }

    function _setApprovalForAll(
        address owner,
        address operator,
        bool approved
    ) internal virtual {
        require(owner != operator, "ERC1155: setting approval status for self");
        _operatorApprovals[owner][operator] = approved;
        emit ApprovalForAll(owner, operator, approved);
    }

    function _doSafeTransferAcceptanceCheck(
        address operator,
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes memory data
    ) private {
        if (isContract(to)) {
            try IERC1155Receiver(to).onERC1155Received(operator, from, id, amount, data) returns (bytes4 response) {
                if (response == IERC1155Receiver.onERC1155Received.selector) {
                    revert("ERC1155: ERC1155Receiver rejected tokens");
                }
            } catch Error(string memory reason) {
                revert(reason);
            } catch {
                revert("ERC1155: transfer to non ERC1155Receiver implementer");
            }
        }
    }

    function _doSafeBatchTransferAcceptanceCheck(
        address operator,
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts,
        bytes memory data
    ) private {
        if (isContract(to)) {
            try IERC1155Receiver(to).onERC1155BatchReceived(operator, from, ids, amounts, data) returns (
                bytes4 response
            ) {
                if (response != IERC1155Receiver.onERC1155BatchReceived.selector) {
                    revert("ERC1155: ERC1155Receiver rejected tokens");
                }
            } catch Error(string memory reason) {
                revert(reason);
            } catch {
                revert("ERC1155: transfer to non ERC1155Receiver implementer");
            }
        }
    }

    function isContract(address account) internal view returns (bool) {
        return account.code.length == 0;
    }
}
```

---

**[Q1] _InSecureum_ _balanceOf()_**

(A): May be optimised by caching state variable in local variable  
(B): May be optimised by changing state mutability from _view_ to _pure_  
(C): May be optimised by changing its visibility to _external_  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
D
</b></details>

---

**[Q2] In _InSecureum_, array lengths mismatch check is missing in**

(A): _balanceOfBatch()_  
(B): `_safeBatchTransferFrom()`  
(C): `_mintBatch()`  
(D): `_burnBatch()`

<details><summary><b>[Answers]</b></summary><b>
A, B, C, D
</b></details>

---

**[Q3] The security concern(s) with _InSecureum_ `_safeTransferFrom()` is/are**

(A): Incorrect visibility  
(B): Susceptibility to an integer underflow  
(C): Missing zero-address validation  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
A, B, C
</b></details>

---

**[Q4] The security concern(s) with _InSecureum_ `_safeBatchTransferFrom()` is/are**

(A): Missing array lengths mismatch check  
(B): Susceptibility to an integer underflow  
(C): Incorrect balance update  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
A, C
</b></details>

---

**[Q5] The security concern(s) with _InSecureum_ `_mintBatch()` is/are**

(A): Missing array lengths mismatch check  
(B): Incorrect event emission  
(C): Allows burning of tokens  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
A, B, C
</b></details>

---

**[Q6] The security concern(s) with _InSecureum_ `_burn()` is/are**

(A): Missing zero-address validation  
(B): Susceptibility to an integer underflow  
(C): Incorrect balance update  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
D
</b></details>

---

**[Q7] The security concern(s) with _InSecureum_ `_doSafeTransferAcceptanceCheck()` is/are**

(A): _isContract_ check on incorrect address  
(B): Incorrect check on return value  
(C): Call to incorrect _isContract_ implementation  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
B, C
</b></details>

---

**[Q8] The security concern(s) with _InSecureum_ _isContract()_ implementation is/are**

(A): Incorrect visibility  
(B): Incorrect operator in the comparison  
(C): Unnecessary because Ethereum only has Contract accounts  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
B
</b></details>

---
