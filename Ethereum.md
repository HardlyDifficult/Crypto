
# Ethereum

## Summary

Ethereum is a trusted [public ledger](about/PublicLedger.md) with support for [smart contracts](about/SmartContracts.md) hosted by [nodes](about/Nodes.md) and secured by [miners](about/Miners.md) using a [Proof of Work algorithm](about/ProofOfWork.md).

When discussing the currency portion of the Ethereum platform (i.e. excluding the smart contracts), it's simply called **Ether**.

### Tech

 - [Docs](http://www.ethdocs.org/en/latest/)
 - POW algorithm: Ethash
    - Mined with GPUs.
    - Changing to POS (sometime in 2018 they hope).
 - ~10 TPS
   - Block size: TODO
   - Difficulty: TODO
     - Recalcs TODO
 - Minting: TODO
    - Decreases TODO
 - Transaction fees
    - Rate defined by supply and demand.
    - 100% to the miner.

#### Solidarity

Solidarity is a programming language written for Ethereum smart contracts. 

 - Solidarity is a clear, simple language, built with the limited capabilities of a smart contract in mind.
    - Pro: security and stability may be easier to achieve.
    - Con: slows adoption as there few developers proficient in the language.

#### Uncles

Ethereum pays miners 7/8th the usual reward for submitting a block which could have been valid if someone else hadn't already created one.

 - Only 2 uncles may be accepted per block.
 - Pro: this limits some centralization of miners by lessening the impact of how quickly you receive updates from the network.
    - For example, Bitcoin has a large percent of miners in China.  There is concern that they have an advantage because when they create a block, Chinese miners are able to start working on the next one before the rest of the world... due to slow connections leaving China.
 - TODO

TODO consensus

### In General 

See [about CryptoCurrency](about/CryptoCurrency.md) about [about SmartContracts](about/SmartContracts.md).  

The comments below are specific to Ethereum and exclude general pros/cons of crypto.

## Market

 - ICOs prefer Ethereum
   - Many ICOs are launched using Ethereum smart contracts.  This drives adoption.
 - Cryptokitties
   - A collectable trading game

### Pros

 - Most active development community (compared to other cryptocurrencies).

### Cons

 - It's a bad name.
   - First couple times I heard the name, I had no idea what to type into google.

## Failures 

 - Ethereum classic
   - There was a smart contract bug which cost people millions.  
     - TODO detail
   - Ethereum created a fork to correct the problem... effectively reversing a smart contact with human intervention.  
     - That's not supposed to be possible!
   - Some of the community refused to go along with the change on principle.  
     - They continue the original fork, now known as "Ethereum Classic".
     - The new fork dominates the market under the name "Ethereum".

## Resources

- https://www.cryptokitties.co 
- [Learn to code by making a zombie trading game](https://cryptozombies.io/)





<br><br><hr>  **Disclaimer**: I am not a financial adviser.  This site includes my thoughts and non-expert opinions.  Do not take action based on what you read here, do your own research and seek professional advice first.

Have a correction or something to add?  Join us daily at [twitch.tv/HardlyDifficult](http://twitch.tv/HardlyDifficult).