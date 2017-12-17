# Proof of Work

## Summary

Proof of work is a race to the bottom security model.  It's about wasting so much resources that it would be stupid for a criminal to try and waste more than the community collectively already is.

### Why Use Proof of Work?

Miners are responsible for protecting against  malicious activity such as double spending (e.g. I use my BTC to buy something from both Bob and Joe, but only one of them can actually get paid.) They construct a block of valid transactions and then use Proof of Work to assert that it is a good block to add to the chain.

### Pros

 - Security with math
    - It's impossible to cut corners, generating blocks using this method is expensive to do.

### Cons

 - Ridiculous amounts of energy consumed
   - Estimated to be about 215 kilowatt-hours (KWh) per transaction. For perspective:
     - My apartment uses 600 KWh per month.
     - The network is currently totaling as much energy as Denmark consumes. 
   - The public, with concerns about the environment, should boycott/protest Bitcoin.
     - This concern is being discussed more and more.
 - Wasteful
    - This method is only effective if the amount of resources spent mining is significantly more than the resources available to an attacker.
 - Centralization
    - The large hardware investment, the noise and heat, and energy costs overtime drive centralization because it becomes harder and harder for a hobbiest to stand a chance.

## How it Works

A miner generates a possible block to add to the block chain.  When doing so, they confirm the transactions included.  They then spend time and money to assert that this block should be added to the chain.

Time and money is spent by way of a guess and check process.  Given the block plus a small blob of data, miners need to find a value for that blob so that a hash of the entire thing starts with a certain number of 0s.

## Security

### How This Creates Security

The chain of blocks that has had the most amount of work put into it is the only chain the network trusts.  If someone malicious created a bad block, the rest of the network should ignore that block.  Overtime the malicious block will be completely overrun by the sheer numbers of people doing the right thing.

Because of this, it is standard to only accept a transaction as final after a few additional blocks have been added to the chain.

### Possible Threats

#### 51% attack

The chain with the most work put into it is the only chain accepted by the network.  If a malicious party is able to acquire 51% of the total mining power, they would effectively have complete control over the chain.

#### Quantum computing

Most Proof of Work algorithms (including Bitcoins) would theoretically be broken overnight by quantum computers. Quantum computers could generate signatures so quickly that a 51% attack becomes very feasible.  







<br><br><hr>  **Disclaimer**: I am not a financial adviser.  This site includes my thoughts and non-expert opinions.  Do not take action based on what you read here, do your own research and seek professional advice first.

Have a correction or something to add?  Join us daily at [twitch.tv/HardlyDifficult](http://twitch.tv/HardlyDifficult).