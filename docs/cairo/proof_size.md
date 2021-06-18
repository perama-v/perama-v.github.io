---
layout: page
title:  "Proof size"
permalink: /cairo/proof-size/
toc: false
---

This page outlines the "space-time" constraints prevalent in scaling discussions. How
are proofs best visualised, and who are the agents that coordinate and create them?

With Cairo programs and their associated STARK proofs, there are different terminologies,
referring to various size- and time-properties. The purpose here is to aggregate these
factors for quick and easy grokking.

## Restating the premise

Ethereum block size is a fixed resource:
- Fixed in-protocol
- Changed by voting during block production
- Large values allow more transactions, but make participation burdensome
- Small values reduce throughput, but make participation practical.
- The social agreement is that an "everyday" computer can participate.

Throughput can be increased by increasing the information density of within a block. A
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
    - STARK proof: 5,000,000 gas (~3 per block).

So a single STARK proof occupies the same space as about 238 ETH transfers. If a proof
represents the 238 ETH transfers, "utility density" is the same as performing those
individual transactions. For a proof representing 1000 transfers, the density is 4x greater.

The **core premise** is that it is preferable to represent transactions at a higher density
because it delivers access to Ethereum to more people.

## Proof costs

**1. Submit and verify proof**

5 million gas for a single proof.

This includes a data storage cost and a cost for registering each fact. The
facts are represented by a merkle tree and the cost is polylogarithmic with
the number of facts. Thus adding new facts adds minimal overhead to the total cost.

The example below shows how a "poly-logarithmic" cost to register facts
changes with increasing batch size (that is, with more facts).
This is the rought structure of a polylogarithmic relationship.

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
added. Practically, each new Cairo program proof adds only a negligible additional cost.

This amortizes the cost of the proof.
[More info here](https://medium.com/starkware/the-fact-registry-a64aafb598b6).

**2. Store new state**

This is a transaction sent to the application contract, which contains data used
for application logic. This may be thought of as fact integration.
For example: "The trading system new total balance is x, with user balances a, b, c, d".
This transaction will incur a minimum cost of 20,000 gas per application.

Consider a proof that represents state updates for 30 applications.
If the applications all decide to immediately update their state over the next few blocks
following the proof, then this would incur a minimum of an additional 600,000 gas (30 * 20,000).

Ideally, these state updates represent many users performing complex L2 operations,
such that the density of the system is greater than that possible with normal transactions.

In this example if the transactions were trades, then the "better than naive L1" threshold
is crossed at 28 trades (`(5,000,000 + 600,000) / 200,000)`) that cost 200,000 gas each.
At 280 trades, the cost is 10x cheaper.

## Proof limits

What are the limits of the system?

The answer lies in the wait time acceptable for applications.

Proofs can be arbitrarily large. Many transactions can be accumulated and the cost per
transaction continues to drop. A rough figure is on the order of ~300 gas per transaction
for simple transfers, as demonstrated in the
[Great Reddit Bake-Off](https://medium.com/starkware/the-great-reddit-bake-off-2020-c93196bad9ce).
The bake-off allowed for a showcase of costs where timeliness was not a restriction.
300,000 transactions were processed, but in what time frame would an app want to verify
that many transactions? Weekly reddit point deliveries might be one example.

However, what if you wanted a state update every day, or every ten minutes? The
time interval between proof delivery is important.

A proof is most dense when allowed to "fill up". Like a bamboo
[鹿威し "shishi-odoshi" (deer scarer)"](https://upload.wikimedia.org/wikipedia/commons/5/52/Shishi_odoshi_2017-02-28.webm)
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
11. The **hour checkpoint** is accepted and the minute checkpoints continue.
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


More information about checkpoints can be found here in this
[ethresear.ch post](https://ethresear.ch/t/checkpoints-for-faster-finality-in-starknet/9633).

## Resource estimates

The gas used for this system design is approximated as follows:

- Ethereum blocks per hour: 257 (60*60/14).
- Total gas available: 3,855 million gas (257*15000000).
- Gas per proof: 5 million gas.
- Gas used by applications updating state (rough estimate for 15 applications with
60,000 gas per transaction): ~1 million gas (15*60000).
- Gas used per hour: 6 million gas.
- Percentage network capacity used: 0.16% (6/3855).

These are napkin calculations only, and they suggest that the system requirements are
a small proportion of the available network capacity. As more StarkNet applications are
deployed and the number of transactions rises, Ethereum block space use will remain
relatively constant. In this way, the cost of using Ethereum for those applications
will drop without compromising the on-chain resource that other users enjoy.
