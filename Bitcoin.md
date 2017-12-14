# Bitcoin

## Summary

The largest and first cryptocurrency.  

Bitcoin is a trusted [public ledger](about/PublicLedger.md) hosted by [nodes](about/Nodes.md) and secured by [miners](about/Miners.md) using a [Proof of Work](about/ProofOfWork.md) algorithm.

### Tech

 - [Whitepaper](https://bitcoin.org/en/bitcoin-paper)
 - ~4 TPS 
   - Block size: 1MB
     - [Segwit](about/Segwit.md) is optional
   - Difficulty: target 10 mins per block
     - Recalcs every 2016 blocks (every 2 weeks)
 - Minting: 12.5 BTC per block to the miner
   - POW algorithm: SHA-256 
     - Mined with ASICs.
   - The amount minted cuts in half every 210,000 blocks and reaches 0 in 2140.
   - TODO next drop date?
 - Transaction fees
   - Rate defined by supply and demand.
   - 100% to the miner.
 - Launched January 2009

### In General 

See [about CryptoCurrency](about/CryptoCurrency.md).  

The comments below are specific to Bitcoin and exclude general pros/cons of crypto.

### Market

 - Filled with Crazy Buzz! 
    - "It's the Money of the Future"
 - Highly volatile
    - +/- 20% in a day is fairly common
 - Futures ([what are futures?](about/Futures.md))
    - Launched Dec 2017, Bitcoin is the first and only with futures (others are already in the works).

## Why Operate a Node?

TODO

### Use Cases

 - Trading Alt Coins
   - Bitcoin is the currency of choice for trading any other cryptocurrency.
 - Large transactions
   - I say "large" because the fees are too much and the speed is too slow for anything but.
   - Cheaper and faster than many wiring options.

### Pros

 - Name Brand of Crypto
 - Secure
   - The arrival of Quantum computers seems to be the only viable attack today.

### Cons

 - Ridiculous amounts of energy consumed
   - Estimated to be about 215 kilowatt-hours (KWh) per transaction. For perspective:
     - My apartment uses 600 KWh per month.
     - The network is currently totaling as much energy as Denmark consumes. 
   - The public, with concerns about the environment, should boycott/protest Bitcoin.
     - This concern is being discussed more and more.
 - At capacity
    - Every block is full.  At the moment, Bitcoin cannot support any more transaction volume than it sees today.
    - There is no official schedule to address this concern.
 - Inability to adapt
   - Bitcoin has no official leaders and no process for adopting new technology or addressing concerns such as scaling.
   - Historically, Bitcoin has not been able to reach consensus on protocol changes quickly.
   - The inability to reach consesus was where Bitcoin Cash and Gold wer born (and why Segwit2x happened and Bitcoin Platum, etc are coming).  These forks only create confusion in the marketplace.
 - Illegal in some countries (https://en.wikipedia.org/wiki/Legality_of_bitcoin_by_country_or_territory)
 - High operating costs
   - Today the new coins minted (12.5 BTC) pay miners about $200,000 per block (every 10 mins)
     - That's almost $30 million per day.
   - Blocks hold about 2k transaction per block: averaging $100 per transaction
   - The numbers above are without the transaction fees, currently averaging (TODO).
 - Very long-term, it's on course to crash to 0 (early 2100s, unless a fork is used to address this concern). 
   - Why: Competition amoung miners is what keeps the network secure.  Miners are compensated for this work by minting new coins with each block.  The number of coins minted decreases periodically.
   - [Read more on bitcoin.org's Myths FAQ](https://en.bitcoin.it/wiki/Myths#After_21_million_coins_are_mined.2C_no_one_will_generate_new_blocks)
 - Miner centralization
   - As compared to other PoW coins, Bitcoin has more centralization of miners due to the specialized hardware available for SHA-256.

### Other Points

 - 51% attack is not viable for Bitcoin
   - The amount of resources being spent on Bitcoin is too big for any entity to take on.  That is not true for other cryptocurrencies.
 - Node Sustainability
   - There is no reward or other direct incentive for operating nodes, and the cost to do so is increasing.
 - Myth: Bitcoin is private
   - No.  There are no names in the system, but every transaction is public and anyone can trace where money comes from and where it goes.

## Resources 

  - https://lopp.net/bitcoin.html

## Failures

 - Mt Gox TODO
 - $60 million stolen from NiceHash
   - Funds are currently being watched by the community with hopes of preventing the attackers from being able to cash out.




<br><br><hr> **Disclaimer**: I am not a financial adviser.  This site includes my thoughts and non-expert opinions.  Do not take action based on what you read here, do your own research and seek professional advice first.

Have a correction or something to add?  Join us daily at [twitch.tv/HardlyDifficult](http://twitch.tv/HardlyDifficult).