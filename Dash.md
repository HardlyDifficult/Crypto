# Dash

## Summary

Dash is a trusted [public ledger](about/PublicLedger.md) hosted by [Masternodes](about/Masternodes.md) and secured by [miners](about/Miners.md) using a [Proof of Work](about/ProofOfWork.md) algorithm.  Private transactions are optional.

## Tech

Features:

 - Governance model
   - 10% minted coins go to an budget reserved by the system.  Masternodes vote on how to project proposals and funds are unlocked for the winners.
 - Instant Transactions 
   - Confirmation in < 1 second.
   - Transactions are validated by a random selection of Masternodes.   
 - Private Transactions (optional)
    - Fee: .0125 DASH
    - Masternodes mix transactions with other random people to mask who is paying who.

Stats:

 - ~12 TPS
   - Block size: 1 MB
   - Difficulty: target 2.5 mins
     - Recalcs every block
 - Minting: 3.60295191 DASH
    - 45% to Miners, 45% to Masternodes, 10% Tech budget
      - POW algorithm: X-11
        - Mined on CPU and GPU
    - Reduces by one-fourteenth every year but never reaches zero.
 - Transaction fees 
  - ~$1 InstantSend
 - Transaction time:
   - < 1 second
 - Launched Feb 2014

### In General 

See [about CryptoCurrency](about/CryptoCurrency.md), [about Proof of Work](about/ProofOfWork.md), and [Masternodes](about/Masternodes.md).  

The comments below are specific to Dash and exclude general pros/cons of crypto.

## Market

 - One of the (if not the) most accepted CryptoCurrency by merchants.

## Pros

 - Fast confirmations (1 second)

## Cons

 - Private transactions rely on trusting the Masternodes.
    - Other coins, such as Monero, have trust built into the platform so your information does not have to go through someone elses hands like this.

## Failures

 - Instamine
   - 10-15% of the total coins which will ever be mined, were mined within the first 48 hours.
   - Their [official response](https://dashpay.atlassian.net/wiki/spaces/OC/pages/19759164/Dash+Instamine+Issue+Clarification)
