**Note**: All 8 questions in this RACE are based on the below contracts. This is the same contracts you will see for all the 8 questions in this RACE. The question is below the shown contracts.

```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity 0.8.17;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";

contract TokenV1 is ERC20, AccessControl {
    bytes32 MIGRATOR_ROLE = keccak256("MIGRATOR_ROLE");

    constructor() ERC20("Token", "TKN") {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
    }

    // Spec wasn't clear about what 'admin functions' need to be capable of.
    // Well, this should do the trick.
    fallback() external {
        if (hasRole(MIGRATOR_ROLE, msg.sender)) {
            (bool success, bytes memory data) = msg.sender.delegatecall(msg.data);
            require(success, "MIGRATION CALL FAILED");
            assembly {
                return(add(data, 32), mload(data))
            }
        }
    }
}

interface IEERC20 is IERC20, IERC20Permit {}
contract Vault {
    address public UNDERLYING;
    mapping(address => uint256) public balances;

    constructor(address token)  {
        UNDERLYING = token;
    }

    function deposit(uint256 amount) external {
        IEERC20(UNDERLYING).transferFrom(msg.sender, address(this), amount);
        balances[msg.sender] += amount;
    }

    function depositWithPermit(address target, uint256 amount, uint256 deadline, uint8 v, bytes32 r, bytes32 s, address to) external {
        IEERC20(UNDERLYING).permit(target, address(this), amount, deadline, v, r, s);
        IEERC20(UNDERLYING).transferFrom(target, address(this), amount);
        balances[to] += amount;
    }

    function withdraw(uint256 amount) external {
        IEERC20(UNDERLYING).transfer(msg.sender, amount);
        balances[msg.sender] -= amount;
    }

    function sweep(address token) external {
        require(UNDERLYING != token, "can't sweep underlying");
        IEERC20(token).transfer(msg.sender, IEERC20(token).balanceOf(address(this)));
    }
}

/* ... some time later ... */

// Adding permit() while maintaining old token balances.
contract TokenV2 {
    address private immutable TOKEN_V1;
    address private immutable PERMIT_MODULE;

    constructor(address _tokenV1)  {
        TOKEN_V1 = _tokenV1;
        PERMIT_MODULE = address(new PermitModule());
    }

    // Abusing migrations as proxy.
    fallback() external {
        (
            bool success,
            bytes memory data
        ) = (address(this) != TOKEN_V1)
          ? TOKEN_V1.call(abi.encodePacked(hex"00000000", msg.data, msg.sender))
          : PERMIT_MODULE.delegatecall(msg.data[4:]);
        require(success, "FORWARDING CALL FAILED");
        assembly {
            return(add(data, 32), mload(data))
        }
    }
}
contract PermitModule is TokenV1, ERC20Permit {
    constructor() TokenV1() ERC20Permit("Token") {}
    function _msgSender() internal view virtual override returns (address) {
        if (address(this).code.length == 0) return super._msgSender(); // normal context during construction
        return address(uint160(bytes20(msg.data[msg.data.length-20:msg.data.length])));
    }
}
```

---

**[Q1] Sensible gas optimization(s) would be**

(A) Making `MIGRATOR_ROLE` state variable constant  
(B) Making `UNDERLYING` state variable constant  
(C) Making `MIGRATOR_ROLE` state variable immutable  
(D) Making `UNDERLYING` state variable immutable

<details><summary><b>[Answers]</b></summary><b>
A, D
</b></details>

---

**[Q2] What would a caller with `MIGRATOR_ROLE` permission be capable of?**

(A): Manipulating TokenV1's storage  
(B): Deleting TokenV1's stored bytecode  
(C): Changing TokenV1's stored bytecode to something different  
(D): With the current code it's not possible for anyone to have `MIGRATOR_ROLE` permission

<details><summary><b>[Answers]</b></summary><b>
A, B
</b></details>

---

**[Q3] Vault initialized with TokenV1 as underlying**

(A): Can be drained by re-entering during withdrawal  
 (B): Can be drained during withdrawal due to an integer underflow  
 (C): Allows stealing approved tokens due to a phantom (i.e. missing) function  
 (D): None of the above

<details><summary><b>[Answers]</b></summary><b>
C
</b></details>    
    
---

**[Q4] If Vault were to use `safeTransferFrom` instead of `transferFrom` then**

(A): It would be able to safely support tokens that don't revert on error  
(B): It would ensure that tokens are only sent to contracts that support handling them  
(C): It would introduce a re-entrancy vulnerability due to receive hooks  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
A
</b></details>    
    
---

**[Q5] Who would need the `MIGRATOR_ROLE` for TokenV2 to function as intended?**

(A): The deployer of the TokenV2 contract  
 (B): The TokenV1 contract  
 (C): The TokenV2 contract  
 (D): The PermitModule contract

<details><summary><b>[Answers]</b></summary><b>
C
</b></details>    
    
---
    
**[Q6] With TokenV2 deployed, a Vault initialized with TokenV1 as underlying**

(A): Is no longer vulnerable in the `depositWithPermit()` function  
 (B): Becomes more vulnerable due to a Double-Entry-Point  
 (C): Stops functioning because TokenV1 has been replaced  
 (D): None of the above

<details><summary><b>[Answers]</b></summary><b>
B
</b></details>    
    
---
    
**[Q7] Vault initialized with TokenV2 as underlying**    
    
(A): Can be drained by re-entering during withdrawal    
(B): Can be drained during withdrawal due to a integer underflow    
(C): Is not vulnerable in the `depositWithPermit()` function    
(D): Is vulnerable due to a Double-Entry-Point     
    
<details><summary><b>[Answers]</b></summary><b>
C, D
</b></details>    
    
---

**[Q8] The PermitModule contract**

(A): Acts as a proxy  
 (B): Acts as an implementation  
 (C): Allows anyone to manipulate TokenV2's balances  
 (D): Can be self-destructed by anyone

<details><summary><b>[Answers]</b></summary><b>
B, D
</b></details>    
    
---
