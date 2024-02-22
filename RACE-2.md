**Note**: All 8 questions in this quiz are based on the _InSecureumDAO_ contract snippet shown below. This is the same contract snippet you will see for all the 8 questions in this quiz. The _InSecureumDAO_ contract snippet illustrates some basic functionality of a Decentralized Autonomous Organization (DAO) which includes the opening of the DAO for memberships, allowing users to join as members by depositing a membership fee, creating proposals for voting, casting votes, etc. Assume that all other functionality (that is not shown or represented by ...) is implemented correctly.

```solidity
pragma solidity 0.8.4;
import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol';
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/Pausable.sol";

contract InSecureumDAO is Pausable, ReentrancyGuard {

    // Assume that all functionality represented by ... below is implemented as expected

    address public admin;
    mapping (address => bool) public members;
    mapping (uint256 => uint8[]) public votes;
    mapping (uint256 => uint8) public winningOutcome;
    uint256 memberCount = 0;
    uint256 membershipFee = 1000;

    modifier onlyWhenOpen() {
        require(address(this).balance > 0, 'InSecureumDAO: This DAO is closed');
        _;
    }

    modifier onlyAdmin() {
        require(msg.sender == admin);
        _;
    }

    modifier voteExists(uint256 _voteId) {
       // Assume this correctly checks if _voteId is present in votes
        ...
        _;
    }

    constructor (address _admin) {
        require(_admin == address(0));
        admin = _admin;
    }

    function openDAO() external payable onlyAdmin {
        // Admin is expected to open DAO by making a notional deposit
        ...
    }

    function join() external payable onlyWhenOpen nonReentrant {
        require(msg.value == membershipFee, 'InSecureumDAO: Incorrect ETH amount');
        members[msg.sender] = true;
        ...
    }

    function createVote(uint256 _voteId, uint8[] memory _possibleOutcomes) external onlyWhenOpen whenNotPaused {
        votes[_voteId] = _possibleOutcomes;
        ...
    }

    function castVote(uint256 _voteId, uint8 _vote) external voteExists(_voteId) onlyWhenOpen whenNotPaused {
        ...
    }

    function getWinningOutcome(uint256 _voteId) public view returns (uint8) {
        // Anyone is allowed to view winning outcome
        ...
        return(winningOutcome[_voteId]);
    }

    function setMembershipFee(uint256 _fee) external onlyAdmin {
        membershipFee = _fee;
    }

    function removeAllMembers() external onlyAdmin {
        delete members[msg.sender];
    }
}
```

---

**[Q1] Based on the comments and code shown in the _InSecureumDAO_ snippet**

(A): DAO is meant to be opened only by the _admin_ by making an Ether deposit to the contract  
(B): DAO can be opened by anyone by making an Ether deposit to the contract  
(C): DAO requires an exact payment of _membershipFee_ to join the DAO  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
A,B,C
</b></details>

---

**[Q2] Based on the code shown in the _InSecureumDAO_ snippet**

(A): Guarded launch via circuit breakers has been implemented correctly for all state modifying functions  
(B): Zero-address check(s) has/have been implemented correctly  
(C): All critical privileged-role functions have events emitted  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
D
</b></details>

---

**[Q3] Reentrancy protection only on _join()_ (assume it’s correctly specified) indicates that**

(A): Only _payable_ functions require this protection because of handling _msg.value_  
(B): _join()_ likely makes untrusted external call(s) but not the other contract functions  
(C): Both A and B  
(D): Neither A nor B

<details><summary><b>[Answers]</b></summary><b>
B
</b></details>

---

**[Q4] Access control on _msg.sender_ for DAO membership is required in**

(A): _createVote()_ to prevent non-members from creating votes  
(B): _castVote()_ to prevent non-members from casting votes  
(C): _getWinningOutcome()_ to prevent non-members from viewing winning outcomes  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
A,B
</b></details>

---

**[Q5] A commit/reveal scheme (a cryptographic primitive that allows one to commit to a chosen value while keeping it hidden from others, with the ability to reveal the committed value later) is relevant for**

(A): _join()_ to not disclose _msg.sender_ while joining the DAO  
(B): _createVote()_ to not disclose the possible outcomes during creation  
(C): _castVote()_ to not disclose the vote being cast  
(D): All the above

<details><summary><b>[Answers]</b></summary><b>
C
</b></details>

---

**[Q6] Security concern(s) from missing input validation(s) is/are present in**

(A): _createVote()_ for duplicate `_voteId`  
(B): _castVote()_ for existing `_voteId`  
(C): _getWinningOutcome()_ for existing `_voteId`  
(D): _setMembershipFee()_ for sanity/threshold checks on `_fee`

<details><summary><b>[Answers]</b></summary><b>
A,D
</b></details>

---

**[Q7] _removeAllMembers()_ function**

(A): Will not work as expected to remove all the members from the DAO  
(B): Will work as expected to remove all the members from the DAO  
(C): Is a critical function missing an event emission  
(D): None of the above

<details><summary><b>[Answers]</b></summary><b>
A,C
</b></details>

---

**[Q8] _InSecureumDAO_ will not be susceptible to something like the 2016 “DAO exploit”**

(A): Because it derives from _ReentrancyGuard.sol_ which protects all contract functions by default  
(B): Only if it does not have a withdraw Ether function vulnerable to reentrancy and makes no external calls  
(C): Because Ethereum protocol was fixed after the DAO exploit to prevent such exploits  
(D): Because Solidity language was fixed after the DAO exploit to prevent such exploits

<details><summary><b>[Answers]</b></summary><b>
B
</b></details>

---
