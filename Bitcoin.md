# Bitcoin

## Summary

The largest and first cryptocurrency.  

Bitcoin is simply a trusted public ledger ([what is a public ledger](about/PublicLedger.md)) secured by miners using [Proof of Work algorithm](about/ProofOfWork.md).

### Tech

 - [Whitepaper](https://bitcoin.org/en/bitcoin-paper)
 - POW algorithm: SHA-256 
   - Mined with ASICs.
 - Block size: 1MB, [Segwit](about/Segwit.md) is optional
 - Difficulty: target 10 mins per block
   - Recalcs every 2016 blocks (every 2 weeks)
 - ~4 TPS 
 - Minting: 12.5 per block to the miner
   - The amount minted cuts in half every 210,000 blocks and reaches 0 in 2140.
 - Transaction fees
   - Rate defined by supply and demand.
   - 100% to the miner.

### Market

 - Filled with Crazy Buzz! 
    - "It's the Money of the Future"
    - Government currencies are going to collapse
    - Freedom!
 - Highly volatile
    - +/- 20% in a day is faily common
 - Futures ([what are futures?](about/Futures.md))
    - Launched Dec 2017, Bitcoin is the first and only with futures (others are already in the works).

### Use Cases

 - Stored value.
   - "Digital Gold" - Bitcoin has value because people think it's going to grow in value.
 - Bypass government
   - Country to country and transactions for any good or service cannot be stopped by any entity.
 - Large transactions
   - I say large, because the fees are too much and speed to slow for anything but.
   - Cheaper and faster than many wiring options.

### Pros

 - Name Brand of Crypto
 - Secure
   - The arrival Quantum computers seems to be the only viable attack today.
 - Predicable/predetermined minting rate
   - Other currencies, such as the USD, have no restrictions on the amount created each year (and often do not even need to disclose the amount added.)  This could lead to inflation.

### Cons

 - Ridiculous amounts energy consumed
   - Estimated to be about 215 kilowatt-hours (KWh) per transaction, for perspective:
     - My apt uses 600 KWh per month.
     - The network currently totaling as much energy as Denmark consumes. 
   - The public, with concerns about the envorinment, should boycott/protest Bitcoin.
     - This concern is being discussed more and more.
 - At capacity
    - Every block is full.  ATM bitcoin cannot support any more transaction volume then it sees today.
    - There is no official schedule to address this concern.
 - Inability to adapt
   - Bitcoin has no official leaders and no process for adopting new technology or addressing concerns such as scaling.
   - Bitcoin has historically not been able to  reach consensus on protocol changes.
   - The inability to reach consesus was where Bitcoin Cash was born (and why Bitcoin Gold, Bitcoin Platum, etc are coming).  These forks only create confusion in the marketplace.
 - Illegal in some countries (https://en.wikipedia.org/wiki/Legality_of_bitcoin_by_country_or_territory)
 - High operating costs
   - Today the new coins minted (12.5 BTC) pays miners about $200,000 per block (every 10 mins)
   - Blocks hold about 2k transaction per block: averaging $100 per transaction
 - Very long term it's on course to crash to 0 (around 2140, unless a hard fork is used to address this concern). 
   - Why: Competition amoung miners is what keeps the network secure.  Miners are compensated for this work by minting new coins with each block.  The number of coins minted decreases periodically and in 2022
   - [Read more on bitcoin.org's Myths FAQ](https://en.bitcoin.it/wiki/Myths#After_21_million_coins_are_mined.2C_no_one_will_generate_new_blocks)
 - Miner centralization
   - As compared to other PoW coins, Bitcoin has more centralization of miners due to the specialized hardware available for SHA-256.

### Other Points

 - Enables illegal trading
   - But one could argue it does enable any more than cash does.  
 - 51% attack is not viable for Bitcoin
   - The amount of resources being spent on Bitcoin is too big for any entity to take on.  That is not true for other cryptocurrencies.
 - Node Sustainability
   - There is no reward or other direct incentive for operating nodes, and the cost to do so is increasing.
 - Myth: Bitcoin is private
   - No.  There are no names in the system, but every transaction is public and anyone can trace where money came from and where it goes.

## Resources 

  - https://lopp.net/bitcoin.html