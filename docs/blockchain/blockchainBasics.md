# Blockchain basics

## Hashes, Nonces & Mining

Hash is a fingerprint of digital data.

In crypto, the hash is normally set to a certain difficulty, e.g. for mining the block, you have to find a hash for the block that starts with 4 zero's (the more zeros the more difficult it is to find), the miner will iterate over the nonce number to find a value that fits with the rest of the blocks data, so that when the hash of the whole block is taken, it starts with 4 zeros.
The miner that finds this value first is normally rewarded with a small bit of crypto.

## Maintaining chain heritage (Democracy!)

Because each blocks data contains the hash of the previous block,
if a previous block is changed, it will change the hash of that block and therefore the `previous` value in the next block after it will change, this will in turn cause the hash value of that block to change too (and therefore not meet the conditions), and all blocks going ahead.

For someone to try to change the value of a blockchain in the middle of the chain, they will have to compute all the blocks going forward too, this will make the chain look valid. However there are some reasons this is not feasible.

1. Blockchain works as a distributed trust system which means the chain that is considered the truth is the one that is the longest + most common amongst the network.
2. To meet the above conditions, one will have to own 51%+ of the network so they can have the most _common_ chain and also this is super computationally expensive as they would have to be mining all the blocks going forward on their 51% owned network to keep up the facade.
