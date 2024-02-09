**Note**: All 8 questions in this quiz are based on the _InSecureum_ contract. This is the same contract you will see for all the 8 questions in this quiz. _InSecureum_ is adapted from a widely used ERC20 contract. The question is below the shown contract.

```solidity
pragma solidity 0.8.10;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/IERC20Metadata.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Context.sol";

contract InSecureum is Context, IERC20, IERC20Metadata {
    mapping(address => uint256) private _balances;
    mapping(address => mapping(address => uint256)) private _allowances;
    uint256 private _totalSupply;
    string private _name;
    string private _symbol;

    constructor(string memory name_, string memory symbol_) {
        _name = name_;
        _symbol = symbol_;
    }

    function name() public view virtual override returns (string memory) {
        return _name;
    }

    function symbol() public view virtual override returns (string memory) {
        return _symbol;
    }

    function decimals() public view virtual override returns (uint8) {
        return 8;
    }

    function totalSupply() public view virtual override returns (uint256) {
        return _totalSupply;
    }

    function balanceOf(address account) public view virtual override returns (uint256) {
        return _balances[account];
    }

    function transfer(address recipient, uint256 amount) public virtual override returns (bool) {
        _transfer(_msgSender(), recipient, amount);
        return true;
    }

    function allowance(address owner, address spender) public view virtual override returns (uint256) {
        return _allowances[owner][spender];
    }

    function approve(address spender, uint256 amount) public virtual override returns (bool) {
        _approve(_msgSender(), spender, amount);
        return true;
    }

    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) public virtual override returns (bool) {
        uint256 currentAllowance = _allowances[_msgSender()][sender];
        if (currentAllowance != type(uint256).max) {
            unchecked {
                _approve(sender, _msgSender(), currentAllowance - amount);
            }
        }
        _transfer(sender, recipient, amount);
        return true;
    }

    function increaseAllowance(address spender, uint256 addedValue) public virtual returns (bool) {
        _approve(_msgSender(), spender, _allowances[_msgSender()][spender] + addedValue);
        return true;
    }

    function decreaseAllowance(address spender, uint256 subtractedValue) public virtual returns (bool) {
        uint256 currentAllowance = _allowances[_msgSender()][spender];
        require(currentAllowance > subtractedValue, "ERC20: decreased allowance below zero");
        _approve(_msgSender(), spender, currentAllowance - subtractedValue);
        return true;
    }
function _transfer(
        address sender,
        address recipient,
        uint256 amount
    ) internal virtual {
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        uint256 senderBalance = _balances[sender];
        require(senderBalance >= amount, "ERC20: transfer amount exceeds balance");
        unchecked {
            _balances[sender] = senderBalance - amount;
        }
        _balances[recipient] += amount;
        emit Transfer(sender, recipient, amount);
    }

    function _mint(address account, uint256 amount) external virtual {
        _totalSupply += amount;
        _balances[account] = amount;
        emit Transfer(address(0), account, amount);
    }

    function _burn(address account, uint256 amount) internal virtual {
        require(account != address(0), "ERC20: burn from the zero address");
        require(_balances[account] >= amount, "ERC20: burn amount exceeds balance");
        unchecked {
            _balances[account] = _balances[account] - amount;
        }
        _totalSupply -= amount;
        emit Transfer(address(0), account, amount);
    }

    function _approve(
        address owner,
        address spender,
        uint256 amount
    ) internal virtual {
        require(spender != address(0), "ERC20: approve from the zero address");
        require(owner != address(0), "ERC20: approve to the zero address");
        _allowances[owner][spender] += amount;
        emit Approval(owner, spender, amount);
    }
}
```

---

**[Q1] _InSecureum_ implements**

(A): Atypical decimals value  
(B): Non-standard _decreaseAllowance_ and _increaseAllowance_  
(C): Non-standard _transfer_  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
A, B
</b></details>

---

**[Q2] In _InSecureum_**

(A): `_decimals()` can have _pure_ state mutability instead of _view_  
(B): `_burn()` can have _external_ visibility instead of _internal_  
(C): `_mint()` should have _internal_ visibility instead of _external_  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
A, C
</b></details>

---

**[Q3] _InSecureum_ _transferFrom()_**

(A): Is susceptible to an integer underflow  
(B): Has an incorrect allowance check  
(C): Has an optimisation indicative of unlimited approvals  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
A, B, C
</b></details>

---

**[Q4] In _InSecureum_**

(A): _increaseAllowance_ is susceptible to an integer overflow  
(B): _decreaseAllowance_ is susceptible to an integer overflow  
(C): _decreaseAllowance_ does not allow reducing allowance to zero  
(D): _decreaseAllowance_ can be optimised with `unchecked{}`  

<details><summary><b>[Answers]</b></summary><b>
C, D
</b></details>

---

**[Q5] _InSecureum_ `_transfer()`**

(A): Is missing a zero-address validation  
(B): Is susceptible to an integer overflow  
(C): Is susceptible to an integer underflow  
(D): None of the above  

<details><summary><b>[Answers]</b></summary><b>
D
</b></details>

---

**[Q6] _InSecureum_ `_mint()`**

(A): Is missing a zero-address validation  
(B): Has an incorrect event emission  
(C): Has an incorrect update of account balance  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
A, C
</b></details>

---

**[Q7] _InSecureum_ `_burn()`**

(A): Is missing a zero-address validation  
(B): Has an incorrect event emission  
(C): Has an incorrect update of account balance  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
B
</b></details>

---

**[Q8] _InSecureum_ `_approve()`**

(A): Is missing a zero-address validation  
(B): Has incorrect error messages  
(C): Has an incorrect update of allowance  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
B, C
</b></details>

---
