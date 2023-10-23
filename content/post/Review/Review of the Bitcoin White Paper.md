---
title: Review of the Bitcoin White Paper
date: 2021-10-24 04:30:53
updated: 2021-10-24 04:30:53
categories: "Review"
tags:
   - "Cryptography"
   - "Bitcoin"
---

This essay is a short report of the white paper – [Bitcoin: A Peer-to-Peer Electronic Cash System](https://bitcoin.org/bitcoin.pdf). It is the Satoshi Nakamoto's well-known publishment that introduces his idea of Bitcoin. The following is an introduction of this system, and my reflections are also included. Not many details are given in this short article, and I encourage everyone who is interested in Block Chain and crypto-currency to read this famous work themselves.

<!--more-->

Bitcoin, a coin system introduced in the paper, is a new distributed ledger system which is designed to correct the problems people found in digital currency. It was early that people found convenient to expend with digital token that have credibility. However, all schemes proposed used to be vulnerable to the double-spending problem or security worries. 

A simple method, like nowadays fiat money, is to build an authority like a government or a bank that everyone trusts. As an example, the company DigiCash proposed a method concerning an unbalanced encode/decode scheme. Each coin issued encode one’s identity, and the owner should be the only one who can easily decode it back. A double-spending will be detected and the criminal will be determined by the bank since the bank can also decode the coin. This sounds good, but inconvenience of user-to-user transactions and the necessity for trust of an authority make it worried for people.

Bitcoin overcomes all these difficulties! In this new system, all coins are considered as a chain of digital signatures, which, like a real-world signature, is a virtual proof of the ownership of someone. A transaction is a transfer of coins, and it is composed of several inputs indicating how much someone wants to pay, and one or two outputs indicating whom to pay for (usually one for payee and one for change). To do a transfer, the original owner has to sign a message including the hash value of the previous transaction and the public key of the next owner, and add the message and signature to the end of the coin. All transactions are published to and recorded by every Bitcoin user , called nodes. Therefore, such chaining structure makes sure that anyone cannot deny their payment in the future, and the security is thus built.

Notice that if a hash collision can be easily made, fraud of transactions will be realized. Therefore, the hash function used in Bitcoin is a cryptographic one, which owns the resistance of collision, i.e., it is considered impossible to find two numbers that have the same hash values. SHA-256 is a famous cryptographic hash. It is estimated by birthday paradox and Poisson approximation that around about $2^{129}$ numbers will the probability of a collision $> 0.5$. This implies an attacker needs to take $10^{24}$ years to deliberately make a collision in today’s computer.

We have not dealt with the double-spending problem, which the Bitcoin system uses a scheme called “block chain” to solve. First, several transactions are combined in a “block”, and each block consists of the hash of the previous one. In this way the history of all happened transactions are recorded, and thus the order of payment is determined. Each new transaction will first be checked with the current block chain, to see whether the same coin was ever paid before it is added into the chain. Once someone tries to double-spend the same coin, each node will find the later spending illegal and deny, as the evidence that this coin was spent earlier is forever recorded as a transaction in some previous block.

What if an attacker tries to modify the chain to erase the record of payment? This is solved by a proof-of-work(PoW) strategy. Each block also consists of a number called “nonce”. This number is set to make the hash of the current block contain many zeros in the beginning. To build a block, each node needs to find a nonce value, which is difficult for a cryptographic hash. The chaining structure of blocks makes it almost impossible to modify any transaction in the history. As one can imagine, any change to previous transactions leads to the change in the block, and all the nonce in this and later blocks need to be re-calculated. All users can therefore put their trusts in the longest chain. Unless someone owns more than half of the computing power, no one can forge a chain beneficial to them.

The computation work for finding the nonce is rewarded. The one who first finds a correct nonce are rewarded with coins. In addition, a regulation specifies the number of zeros required in the hash increases when blocks are generated too fast recently. This makes sure that there are always people willing to do the computation efforts, and the inflation is also controlled. Another advantage of the scheme is that an attacker having strong computation power finds it more profitable to follow the rules rather than to commit the fraud.

To sum all, the Bitcoin system proposed in this paper and all its later works provide a new frontier for future transactions and sharing economy. It is another revolutionary application of cryptography that makes our lives more convenient. In the last decade, several issues concerning the original Bitcoin system arise, and many solutions and new rules designed to deal with them comes one after another. From my perspective, there is no system able to serve all needs eternally. Following news and [BIPs](https://github.com/bitcoin/bips)(Bitcoin Improvement Proposals) is at least what everyone can do to help our society improve.
