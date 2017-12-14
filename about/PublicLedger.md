# Public Ledger

## Summary

A public ledger is a series of events, similar to rows in a database entered in sequence.

The ledger is used to track transactions where quantity of coin was sent from one address to another. 

## Security

Balances are tracked by 'wallet', which is a 256-bit address.  The wallet address is public (like your email address) and there is an associated private key (which acts like your password).  That private key is used by the owner of the wallet to create signatures.  Signatures are a way of proving that you know the private key and are authorizing a communication, without actually sharing that key.

In order for a transaction to be added to the ledger, it must have a valid signature from the owner of the wallet spending money (no confirmation is required to receive funds.)