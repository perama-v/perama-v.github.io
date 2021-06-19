---
layout: page
title:  "Proof size"
permalink: /cairo/proof-size/
toc: false
---

This page explores proof sizes, how they are important for scaling discussions and how
they relate to the proofs produced by Cairo applications. This post only explores rollups,
where all layer 2 transaction data is available on-chain.

## Restating the premise

Ethereum block size is a fixed resource:
- Fixed in-protocol.
- Changed by voting during block production.
- Large values allow more transactions, but make participation burdensome.
- Small values reduce throughput, but make participation practical.
- The social agreement is that an "everyday" computer can participate.

Throughput can be increased by increasing the information density within a block. A
transaction that is highly compressed and represents the final outcome from some off-chain
agreement is desirable because it allows other transactions to fit.

Parameters:

- Block size: 15 million gas.
- Block time: 14 seconds.
- Transaction sizes:
    - ETH transfer: 21,000 gas (~700 per block).
    - Token transfer: 30-50,000 gas (~300 per block).
    - Token trade: 100-130,000 gas (~115 per block).
    - Mint NFT, lend, borrow : 150-200,000 gas (~75 per block).
    - Tornado cash deposit: 950,000 gas (~15 per block).

STARK Proof size:

- Given an estimated [1100 gas per transaction](https://twitter.com/StarkWareLtd/status/1405133782118809602).
- With a STARK proof approximately [5,000,000 gas per proof](https://ethresear.ch/t/checkpoints-for-faster-finality-in-starknet/9633)
(~3 per block).
- A proof this size might represent around 4500 transactions (~13,500 per block).

So a single STARK proof occupies the same space as about 238 ETH transfers. If a proof
represents the 238 ETH transfers, "utility density" is the same as performing those
individual transactions. For a proof representing 4500 transfers, the density is 18x greater.

The **core premise** is that it is preferable to represent transactions at a higher density
because it delivers access to Ethereum to more people.

## Proof costs

**1. Submit and verify proof**

The size of a proof is practically proportional to the number of transactions it contains.

Different types of transactions may be represented in different degrees of efficiency. An
aggregate of simple transfers may be organised more efficiently for proving than a collection
of non-homogeneous transaction types. Practical "in-production" measures are the truest test,
and 5 million gas proofs, at 1100 gas per transaction are the approximate current measures.

An estimate for how much much smaller transaction data can be represented places transactions
at [10x-100x smaller size](https://vitalik.ca/general/2021/01/05/rollup.html). Trades
on Ethereum cost around 100,000 gas, so the 1100 gas per trade real-world measure
of StarkEx and Deversi [trades](https://twitter.com/StarkWareLtd/status/1405133782118809602)
place on-chain in-production STARK proof compression at around ~100x.

5 million gas for a single proof.

This includes a data storage cost and a cost for registering each fact. The
facts are represented by a merkle tree and the cost is poly-logarithmic with
the number of facts. Thus adding new facts adds minimal overhead to the total cost.

The example below shows how a "poly-logarithmic" cost to register facts
changes with increasing batch size (that is, with more facts).
This is the rough structure of a poly-logarithmic relationship.

```
cost = log(facts)^constant
```
Some example-only (arbitrary) numbers to demonstrate the relationship:
```
# fact = 100000, constant = 5
cost = log(100000)^5 = 5^5 = 3125 "cost units"

# For "n" times as many facts
cost = log(n * facts)^constant
cost = (log(n) + log(facts))^constant  #  Separate the log.
```
Increasing the batch size by 1000x increases the proof size by a far less extent.
```
# 2x as many facts (n = 2), only 1.3x the cost.
cost = (log(2) + log(100000))^5 = (0.3 + 5)^5 = 4181 "cost units"

# 20x as many facts (n = 20), only 3x the cost.
cost = (log(20) + log(100000))^5 = (1.3 + 5)^5 = 9924 "cost units"

# 1000x as many facts (n = 1000), only 10.4x the cost.
cost = (log(1000) + log(100000))^5 = (3 + 5)^5 = 32768 "cost units"
```

This basic example shows that the cost for every additional fact drops as more facts are
added. Practically, each new Cairo program proof adds only a negligible additional verification
cost. The cost of verification is shared across the different applications, more
details on how this works can be found
[here](https://medium.com/starkware/the-fact-registry-a64aafb598b6).

**2. Store new state**

Every fact must be integrated into the smart contract of the application that will use it.
This requires a transaction sent to the application contract, which contains data used
for application logic. The data is corroborated with the Verfier contract before
storage and integration.
For example: "The trading system new total balance is x, with user balances a, b, c and d".
This transaction will incur a minimum cost of 20,000 gas per application.

Consider a proof that represents state updates for 30 applications.
If the applications all decide to immediately update their state over the next few blocks
following the proof, then this would incur a minimum of an additional 600,000 gas (30 * 20,000).

Ideally, these state updates represent many users performing complex L2 operations,
such that the density of the system is greater than that possible with normal transactions.

A STARK rollup is more dense than mainnnet when the proof costs and the costs to integrate
facts are, in total, less than the mainnet transactions they replace. For 100,000 gas trades
being replaced by a 5 million gas proof with 600,000 gas fact integration costs, the
crossver value is `(5,000,000 + 600,000) / 200,000) = 56 trades`. For more than 56 trades, it is
more efficient to use the rollup.

## Proof limits

What are the limits of the system?

The answer lies in the wait time acceptable for applications.

Proofs can be arbitrarily large. Many transactions can be accumulated and the cost per
transaction continues to drop. An example is the 94 million gas used to prove 300,000
simple transfers for the
[Great Reddit Bake-Off](https://medium.com/starkware/the-great-reddit-bake-off-2020-c93196bad9ce).
The bake-off allowed for a showcase of costs where timeliness was not a restriction.
Transactions were only ~300 gas each and were designed to represent the summary for one week's
worth of reddit community points. It would be a different scenario to want to process
300,000 margin trades, which have greater time restrictions.

What is the ideal inter-proof interval? Every day, or every ten minutes?

A proof is most dense when allowed to "fill up". Like a bamboo
[鹿威し "shishi-odoshi" (deer scarer)](https://upload.wikimedia.org/wikipedia/commons/5/52/Shishi_odoshi_2017-02-28.webm)
with a large bore, as the size of a proof grows, the longer it takes to fill. If apps can
wait for twice as long, they can enjoy cheaper fees. Creating proofs at twice the
frequency (like a smaller, faster, shishi-odoshi), halves the number of transactions per proof.
That translates to less efficient Ethereum block space usage and higher fees for the L2 users.

What can be done? Checkpoints.

## Checkpoints

Proofs can be handed to users before their proof has been sent to Ethereum.

The guarantee they they have is that the proof has been integrated into the growing
proof-blob, a structure that accumulates and integrates smaller proofs until the moment
it lands on chain.

Checkpoints are a proof that a transaction is "in the shishi-odoshi". With patience,
the Ethereum proof will become available. Until then, there is no undoing of the checkpoint,
which is an append-only proof.

The process for constructing a checkpoint is as follows:
1. StarkNet nodes begin with a single proof.
2. A new transaction is generated by a user.
3. The transaction is shared with other nodes.
4. A operator called the **Sequencer** detects the transaction. In the Constellations phase
this is a single operator. Int the Universe phase, the role is appointed by leader election.
5. The Sequencer collects other new transactions and decides how to order them.
6. Every minute, the Sequencer passes the transactions to an operator called the **Prover**.
7. The Prover creates a proof and passes it to the other nodes with data (E.g., inputs).
8. The nodes verify the proof and store it and important data as a **minute checkpoint**.
9. The Sequencer and Prover repeat their tasks, this time linking the proof to the previous
proof, forming a **chain of checkpoint proofs**. This is a recursive proof.
10. Once per hour, the proof is submitted to Ethereum (the "shishi-odoshi" clunks).
11. The **hour checkpoint** transaction is accepted and the minute checkpoints continue.

Failsafe scenario.

12. A new user creates a transaction.
13. The user records the checkpoint proof that their transaction was incorporated into.
14. They execute another transaction that depends on the first transaction.
15. A **crisis** occurs and their proof never goes to Ethereum.
16. The user, highly motivated, submits their checkpoint proof to Ethereum.
17. The proof is accepted by the Verifier contract (the user incurs the gas fee).
18. All facts present in that checkpoint proof are verified.
19. The StarkNet nodes resolve the crisis and resume their operations.
20. Users continue to make use of minute checkpoints to for convenient, secure and
optimistically low-cost advantage.

## Resource estimates

The gas used for this system design is approximated as follows:

Assumptions:

- Ethereum blocks per hour: 257 (60*60/14).
- Total gas available: 3,855 million gas (257*15000000).
- Gas per proof: 5 million gas.
- Gas used by applications updating state (rough estimate for 15 applications with
60,000 gas per transaction): ~1 million gas (15*60000).
- Gas used per hour: 6 million gas.

Estimate:

- Percentage network capacity used: 0.16% (6/3855).

These are napkin calculations only, and they suggest that the system requirements are
a small proportion of the available network capacity. As more StarkNet applications are
deployed and the number of transactions rises, Ethereum block space use will remain
relatively constant. In this way, the cost of using Ethereum for those applications
will drop without compromising the on-chain resource that other users enjoy.
