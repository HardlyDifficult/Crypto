# Proof of Work

## Summary

Proof of work is a race to the bottom security model.  

### Pros

 - Math

### Cons

 - Wasteful
 - Centralization

## How it Works

A miner generates a possible block to add to the block chain.  When doing so, they confirm the transactions included.  They then spend time and money to assert that this block should be added to the chain.

Time and money is spent by way of a guess and check process.  Given the block plus a small blob of data, miners need to find a value for that blob so that a hash of the entire thing starts with a certain number of 0s.

## Security

### How This Creates Security

The chain of blocks that has had the most amount of work put into it is the only chain the network trusts.  If someone malicious created a bad block, the rest of the network should ignore that block.  Overtime the malicious block will be completely overrun by the sheer numbers of people doing the right thing.

### Threat Model

51% attack

Quantum computing
