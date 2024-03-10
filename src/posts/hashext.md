---
title: Understanding Hash Length Extension Attacks
date: 2024-03-06 00:00:00
tags:
 - crypto
---

In a recent Hack the Box challenge the solution require a hash length extension attack to alter a session token and give a user admin access. I found the idea of this fascinating and wanted to take a look into the implementation of this instead of just using an existing tool. I found [this implementation and explantion](https://github.com/iagox86/hash_extender/tree/master) extremely useful in understanding the attack and in highlighting differences between different hashing algorithms.

## The Vulnerability

The attack is made possible by a vulnerability in the [Merkle-Damgård hash function](https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction) which is used by a number of hashing algorithms including MD5 and SHA-2. The hashing algorithm will pad the text to be a multiple of the block length and using either the IV (for the first block) or the state from the last bock will generate a new state. This state is then added to the previous one and this is repeated for every block.

This means we can extend the hash as the output we get is just the internal state, we can manually set this state and add more data to get a new hash. To generate the corresponding text we need to pad our input text and then append our added data.

Note that this attack is possible with an unknown salt as this simply changes the offsets of how much we need to pad and the salt will be added back when the server is checking our data.

## Implementing the Attack

While the afformentioned implementation is perfectly functional, I wanted to write my own to make sure I understood the attack fully. Luckily with the power of [Zig](https://ziglang.org/) I was able to write this generically and it worked first time for all other applicable hashing algorithms in the standard library.

[See the source here](https://github.com/grailofpwn/exthash).
