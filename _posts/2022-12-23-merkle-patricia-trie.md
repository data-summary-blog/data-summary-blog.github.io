---
layout: post
title: "Blockchain Review: Merkle Patricia Trie"
categories: Blockchain
author: "Yongchan Hong"
---

# Merkle Patricia Trie
Ethereum is one giant state machine, and uses data storage to record states. 
Each state will be made up of "account", which contains nonce, ether balance, contract code, storage (Account will be explained in later reviews). The state will be changed after transactions.
In order to save all these states, the most ideal way is to use key-value store. In this case, key will be 32 byte address (hash), and value will be a content.
The most efficient key value storage that Ethereuem chose was `Modified Merkle Patricia Trie`.
Let us first dive into Merkle Tree, Patricia Trie, and Modified Merkle Patricia Trie.

> Why don't we use simple key-value storage?
There are more than 100 million accounts in Ethereum. Hashing all those accounts every time will be very inefficient.

## Patricia Trie
![](https://miro.medium.com/max/640/0*t6uY2JJbTUyRZMfg)  
Patricia Trie is a data structure that uses key as a path: same prefix share same path. This is extremely useful when finding common prefixes. Trie will have Big O of O(n) for both search and insertion.  

## Merkle Tree
![](https://miro.medium.com/max/720/0*mrGXIWdQp6n9WT6g)  
Merkle Tree is a tree of hashes with leaf node storing data. Top node will sum children hash and store it. This is extremely useful when state change - you can add data to last tree (specifically tree containing account) and only append to node.  
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99E5573F5AF3BDA013)  

## Merkle Patricia Trie
![](https://devkly.com/static/3352c87d8606fe8816a798f158c37136/c1b63/MPT.png)  
In Merkle Patricia Trie, there are leaf node, extension node and branch node. `Leaf node` will not have child node, and consists path and value. Value will contain information about the address. `Branch node` will have 16 branches and 1 node values. This will consist child nodes. Finally, there is `extension node`, which is an optimized node of branch node. When branch node has only one child node, this will be compressed to extension node that has a path and the hash of the child.  
Since both leaf node and extension node has an array of two items, we use prefix to divide them. As shown in image, extension node will have prefix of 0, 1 and leaf node will have prefix of 2, 3. The number will based on even/odd of nibbles.

> Nibble is the unit used to distinguish key values, and each nibble will be 4 bit. To simplify, each even/odd nibbles is equivalent to path lengths.

In next chapter, we will review three trie: World State Trie, Transaction Trie, and Receipt Trie.


### Reference
https://medium.com/codechain/modified-merkle-patricia-trie-how-ethereum-saves-a-state-e6d7555078dd
https://devkly.com/blockchain/ethereum-state/
https://medium.com/@eiki1212/ethereum-state-trie-architecture-explained-a30237009d4e
https://hamait.tistory.com/959
