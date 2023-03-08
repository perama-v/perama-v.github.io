---
layout: page
title:  "HemiDemiSemiArchive Node"
permalink: /ethereum/protocol/archive
toc: true
---
```
A node in a tree;
What lies inside? Can I try
with scissors, and tape?
```
TL;DR: Snip up an archive node and expose it trustlessly amongst Portal Network nodes
for light weight transaction tracing.

Challenges faced:
- Tracing a transaction requires executing preceeding transactions in that block
    - A "distributed trace transaction" must be a wrapper on a "distributed trace block".
- Block state requires contract code for contracts accessed
    - A "distributed trace block" requires contract bytecode to be available for state proofs,
    which effectively amplifies bytecode compared to a regular archive node.

Interesting facets:
- We really can distribute/shard archive nodes if slow/non-contiguous tracing is of interest.
- A distributed archive node can be multiples more disk size than the theoretical minimum but
individual users will not experience that pain.

Associated experimental rust library: https://github.com/perama-v/archors

---
Table of Contents.

- [Structured (EIP-like): A JSON-RPC method to enable disctributed archive nodes.](#structured-eip-like-a-json-rpc-method-to-enable-disctributed-archive-nodes)
  - [Abstract](#abstract)
  - [Motivation](#motivation)
  - [Specification](#specification)
    - [Description](#description)
    - [Parameters](#parameters)
    - [Returns](#returns)
    - [Example](#example)
  - [Rationale](#rationale)
  - [Backwards Compatibility](#backwards-compatibility)
  - [Test Cases](#test-cases)
  - [Reference Implementation](#reference-implementation)
  - [Security Considerations](#security-considerations)
  - [Copyright](#copyright)
- [Unstructured (blog-like): A look at what is involved in enabling disctributed archive nodes.](#unstructured-blog-like-a-look-at-what-is-involved-in-enabling-disctributed-archive-nodes)
  - [Recap](#recap)
  - [Terminal goal](#terminal-goal)
  - [What is an archive node?](#what-is-an-archive-node)
  - [What function is desired?](#what-function-is-desired)
  - [Stated goals](#stated-goals)
  - [Being frank about disk inefficiency](#being-frank-about-disk-inefficiency)
  - [Transaction proofs](#transaction-proofs)
  - [Minimum proving requirements for a transaction trace](#minimum-proving-requirements-for-a-transaction-trace)
  - [The absence of inter-transaction state roots](#the-absence-of-inter-transaction-state-roots)
  - [Comparison to proofs of EVM execution](#comparison-to-proofs-of-evm-execution)
  - [`trace_transaction` from `trace_block`](#trace_transaction-from-trace_block)
  - [State proofs as a database](#state-proofs-as-a-database)
  - [Proof creation](#proof-creation)
  - [State proof size estimates](#state-proof-size-estimates)
  - [Total state proof burden](#total-state-proof-burden)
  - [On the appetite for access](#on-the-appetite-for-access)
  - [Why not IPFS](#why-not-ipfs)
  - [Contrast to an archive node databases](#contrast-to-an-archive-node-databases)
  - [Example: Looking for state in a trace](#example-looking-for-state-in-a-trace)
  - [Simple transfer exploration: Using `debug_traceTransaction` with `prestateTracer`](#simple-transfer-exploration-using-debug_tracetransaction-with-prestatetracer)
  - [Verifying the preStateChange result of a simple transfer](#verifying-the-prestatechange-result-of-a-simple-transfer)
  - [Basic contract interaction exploration: Using `debug_traceTransaction` with `prestateTracer`](#basic-contract-interaction-exploration-using-debug_tracetransaction-with-prestatetracer)
  - [Summary of `debug_traceTransaction` explorations](#summary-of-debug_tracetransaction-explorations)
  - [Modifications to `prestateTracer` for proofs](#modifications-to-prestatetracer-for-proofs)
  - [On contract bytecode duplication](#on-contract-bytecode-duplication)
  - [debug_traceTransaction for slots, eth_getProof for slot proofs](#debug_tracetransaction-for-slots-eth_getproof-for-slot-proofs)
  - [State](#state)
  - [Validation](#validation)
  - [Integration](#integration)

---
# Structured (EIP-like): A JSON-RPC method to enable disctributed archive nodes.

Presented as an EIP for fun. If an actual EIP is made, it will be noted here
```
title: eth_getArchiveBlockStateProof JSON-RPC method
description: A method for archive nodes to horizontally scale by
providing provable state values required for isolated block tracing.
author: @perama-v
discussions-to: -
status: -
type: Standards
category: Interface
created: 2023-02-28
```

## Abstract

A JSON-RPC method that returns a tree of data that is necessary to execute the transactions
in a block. A client with the block can use the data to re-execute transactions to
with behaviour equal to `trace_transaction`. A network of nodes that distribute
these data can collectively serve as a horizontally scaled archive node, allowing nodes
with disk sizes 0.1-10% of a regular archive node.

The tree is rooted in the block state root and a Merkle proof of the tree
values is used to confirm the correctness of the tree. After an initial construction of
the data for the full blockchain by an archive node, a non-archive node can maintain the
chain tip by constructing the data before blocks become 128 blocks old (~1hr).

## Motivation

Historical (older than 128 blocks) Ethereum transactions are useful for understanding
the complete behaviour of past activity. This information is either available by
running a disk-heavy archive node (>1TB) that stores intermediate chain states, or by
asking someone who does (trusted third party).

To re-execute any historical transaction the `debug_traceTransaction` or `trace_transaction`
endpoints are required. Full nodes cannot provide these data as after they have verified the
transaction, they forget the state values that existed before a transaction modified them.
In other words, the full node method `eth_getTransactionByHash` provides enough information
to begin replaying a transaction, but not to complete it.

The creation and distribution of a small database for each block solves this problem.
A block obtained from a full node via `eth_getBlockByHash` contains the state root and the
unapplied transactions. When the transactions are executed in order, whenever a state value
is required (such as read storage from a contract) this database can be used. The data
is the value that existed in that contract at that historical point in time, a fact that is
guaranteed by the database being a merkle proof rooted in the state root of that block.

A node can verify block canonicality without storing all block headers through use of
a double-batched merkle log accumulator (not covered here).

The outcome is that a user can have a client participating in a network that distributes
block state proofs.

1. User calls `trace_transaction` with a transaction hash.
2. The node consults an index to determine the block hash of the block the transaction is from.
3. Node calls `eth_getBlockByHash` which gets all transactions in the block.
4. Node calls `eth_getArchiveBlockStateProof` which gets the state values the transactions access.
5. Node replays all transactions prior to the specified transaction.
6. Node replays the specific transaction and returns the full trace in a specified style
("vmTrace", "trace", "stateDiff").
7. The transaction can be used or displayed as relevant.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted
as described in RFC 2119 and RFC 8174.

### Description

Returns the data representing state that the specified block requires access to for execution.
The data MUST contain a tree whose root is the state root contained in the block header.

For required data that is not part of the state tree, the data MUST contain objects representing
this ancillary state (such as a preceeding block hash, with a proof against an accumulator).

### Parameters

|Number|Type|Description|
|-|-|-|
|1|{Data}|hash of a block|

### Returns

`{null|object}` - null if no state proof is found, otherwise a state proof object:
- Block state:
    - TBD: Proof data for all state accessed during the block transactions.
- Ancillary state:
    - TBD: Data not rooted in the current block state, such as block headers with proofs
    (against a block header accumulator) of any of the prior 256 blocks.

### Example
Message
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "eth_getArchiveBlockStateProof",
  "params": [
    "0xffe3e973f7b4dae3609675c627826bbc01962a88ed6ba8251e01586ed9be6493"
  ]
}
```
Response
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": {
    "blockStateProof": [
      "TODO - Merkle poof against header state root for all first-accessed state in a block",
    ],
    "ancillaryStateProof": [
      "TODO - Merkle proof against accumulator for block headers used by BLOCKHASH opcode",
    ]
  }
}
```

## Rationale

Regular archive nodes and APIs will continue to be useful for some users. The design allows
for infrequent examination of specific transactions, such as interrogating interesting transactions
or performing accounting a specific wallet address.

The RPC method cannot be applied at the level of the transaction because EIP-658 removed the
post-transaction state root from transactions. Hence the data is rooted to the block, and
preceeding transactions must be re-executed.

This EIP describes an RPC method as a coordination point between clients who wish to opt in to
supporting such a use case. The proof data could be created and distributed via any
content distribution network but integration into a node is more useful because 1) proofs
are only useful in the context of the blocks 2) the data will grow over time and a
node client is well suited to facilitate this.

This EIP is created with the intent that a Discovery protocol overlay network could exist to
provide archival state. A protocol such as the Portal Network could be extended an archival
state network. This would extend the function of Portal network nodes with low integration
cost. Portal nodes could opt in to access (and provide) `trace_transaction` as required.

The association of a transaction hash to a block hash requires an index, the design of which
is not covered here. As this applies to a horizontally scaled full node, such an index could
be reused.

## Backwards Compatibility

No backward compatibility issues found.

## Test Cases

The following TODOs are noted:
- State transition correctness: Get a block, trace the transactions, record the values accessed
in state, represent as tree. Use that tree to recompute the post-block state, verify it matches
the state root of the next block.

## Reference Implementation

The following TODOs are noted:
- Creator: Write a method responds to `eth_getArchiveBlockStateProof` that calls `trace_block`,
receives the trace, then parses the trace to construct a tree. Finally, returning the
tree to the callee (or storing for later distribution).
- Consumer: Write a method that responds to `eth_getArchiveBlockStateProof` that calls peers in
an overlay network, requesting the state proof tree. Once received, the proof is checked
against the known state root for that block.
- Executor: Obtain a block (`eth_getBlockByHash`) and then executes the block using a
state proof tree for data when state values are accessed (read/write).

## Security Considerations

Some considerations are:
- Blocks may access block hashes from preceeding 256 blocks, but this is not in the
state root. How should this be provided? Perhaps as an additional tree per historical block?
- Network denial of service attacks. Proof sizes may be up to ~36MB and requesting
data could represent an attack on peers serving data. Mitigations include:
    - Use of strict peer rejection procedures or request limits.
- Block canonicality for pre-merge blocks is ensured by an accumulator that can be hard coded
into the client. For post-merge blocks, a beacon chain light client contains an accumulator that
must be used.
- Delayed proof construction. If at the chain tip there is a 1 hour period where nodes fail
to produce block state proofs then an archive node must be used. This constitutes a risk
of having recent blocks be unavailable if an archive node is not integrated with the network.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).

# Unstructured (blog-like): A look at what is involved in enabling disctributed archive nodes.

## Recap
We explored in the last [post series](./poking.md) how one could have an historical view
of your own wallet acvitity with less than 1 GB. The secret was to distribute data between
peers. This invovled:
- Splitting the Unchained Index (in a different way than it currenly is) to locate transactions
- Splitting an Ethereum full node (this is the function of the Portal Network) to access old transactions
- Splitting up a database of 4-byte signatures to decode events
- Splitting up a nametag database to label contracts


What else can we split up? What about an ARCHIVE NODE split into tiny parts?

Let's call it HemiDemiSemiArchive node for short.

## Terminal goal

Look closely at an Ethereum transaction.
```
> Someone tells you that a hack went down and sends you a transaction hash
```
A transaction hash! Great, with a Portal Node (~= distributed full node)
you can fetch the full transaction and look at data including:
- Data sent to the transaction
- Logs that were emitted during the transaction

```
> Oh, maybe the hacker did something unusual and didn't log it...
```
This is not present in a transaction receipt. A transaction trace is needed, which
only an archive node can provide. Running an archive node takes up >1TB, so this
could be a big ask for a single transaction.
```
> Why can't we split up an archive node like we do a portal node?
```
It's a problem of recursion. Transactions calling contracts, calling contracts, calling
contracts... If you are relying on peers to look up what happened, it may take a long time
to follow the rabbit hole of a deeply nested and complicated transaction.
```
> Is it possible? How complicated would it be?
```
Let's find out together.
```
> What might it look like?
```

You start a HemiDemiSemiArchive node. It starts finding peers
and collecting a subset of all the archive data.

You call `trace_transaction`, requesting the full trace of the transaction you have the hash for.

The node checks locally first, then then starts asking peers - locating the
peers that are supposed to have the block containing that particular transaction.

The peer is found and the state in that block allows you to reconstruct the entire block -
including the hack-transaction in question.

You pipe the transaction trace to a local visualiser and start your analysis. Every part
of the EVM execution is available for inspection.

In another scenario you could use TrueBlocks Unchained Index to work out which transactions
are relevant to your wallet/contract. You could then request those specific transactions
for inspection/accounting/analysis.

## What is an archive node?

A quick refresher:
- A full node can generate an archive node (by self-introspecting and expanding it's database)
- A full node keeps record of every transaction that happened
- Once a full node processes a transaction, it stores the new state and forgets the old one
- An archive node does not forget these older states.

## What function is desired?

Here are things that an Archive node can uniquely provide divided in to things that
a HemiDemiSemiArchive node user might desire and obtain:

- Desirable:
    - `trace_transaction`
        - A trace of a specific transaction
    - `trace_replayTransaction`
        - Like `trace_transaction` ("trace") but offers "vmTrace", and "stateDiff" too.
- Possibly desirable:
    - `trace_call`
        - A trace following a message call (from x to y with data z)
    - `trace_callMany`
        - Multiple sequential trace calls. May be infeasible.
    - `trace_rawTransaction`
        - A trace of a simulated transaction
- Desirable if it is a more effective means to get `trace_transaction`
    - `trace_block`
        - Might be too big for scope. Gets all transaction traces in a block.
    - `trace_replayBlockTransactions`
        - Like `trace_block` ("trace") but offers "vmTrace", and "stateDiff" too.
- Likely not suited to HemiDemiSemiArchive nodes:
    - `trace_filter`
        - Is too big for scope. Get all blocks in range that meet sender/recipient criteria.

Thoughts on `trace_block` vs `trace_transaction`. I am currently thinking that a HemiDemiSemiArchive
node should provide one OR the other.

## Stated goals

Given the above calls, perhaps we should define a HemiDemiSemiArchive node as:
- Relies on peers for data
- Provides data to peers
- Provides `trace_transaction`
    - Likely via `trace_block`
- Tunable disk size

## Being frank about disk inefficiency

A HemiDemiSemiArchive node would likely lose a lot of efficiency.
Suppose that the data occupied a multiple of the original when split
up for distribution. What multiple would be unacceptable?

Consider a network where the default node size is set to 10GB.
How many nodes do you need to represent that data?

|Multiple|Size|Need x nodes with 10GB|
|-|-|-|
|1x|2TB|viable with 200 nodes|
|2x|4TB|400 nodes|
|5x|10TB|1000 nodes|
|10x|20TB|2000 nodes|

10GB is an arbitrary value, but it highlights how
database inefficiency
puts strain on the network partipation requirements, rather
than on the individual user.

In current archive node land users are very sensitive to changes that cross mutiples of hard drive
size. (1TB, 2TB, 4TB), So a database that grows slightly above 2TB
could knock out many users. In a distributed archive node system database growth roughly
equates to needing more nodes.

## Transaction proofs

A block header contains a transactions root. A merkle tree with transaction hashes as leaves
allows a proof that a specific transaction is part of a block.
If a peer has a canonical block header (e.g., from an accumulator)
then they can accept a proof that a transaction belongs to that block.

A transaction is a program to be run, and so having a transaction does not tell you what
the transaction effect was. A transaction receipt contains some (but not all) effects that
the transaction had. This includes the gas used by a transaction, and any logs or events emitted.
A block header also has a receipts root, and so a receipt proof can be constructed.

|Kind|Represents|Example|Rooted in header?|Hash and prove?|
|-|-|-|-|-|
|Transaction|The program to be run|Call contract x with y data|Yes|Yes|
|Transaction receipt|Important program outcomes|Tx succeeded, used x gas, emitted y event|Yes|Yes|
|Transaction trace|Every step of the program|Each step has an opcode and counter/stack/memory/storage/gas|No|**No**|

A proof for a transaction trace cannot be built in the same way as the transaction or receipt. There is no special transaction trace format that is hashed and stored somewhere
in the block. This is mostly because it would be a waste of space, one can re-run
a transaction to get the trace.

## Minimum proving requirements for a transaction trace

A transaction trace consists of everything a transaction did. This includes
reading from and writing to storage. After a transaction is run, any state (storage) changes
are saved, but they are not recorded anywhere until the end of the block.
They used to be stored as part of the transaction (post-transaction state root), but
this was removed in the Metropolis hard fork via EIP-658.

To prove a transaction trace you need:
- The transaction (program to be run), provable against the transactions root hash.
- The state at the start of the transaction, which can be found by:
    - Having the state at the start of the block (storage values), provable agaisnt the state root hash.
    - Applying the transactions that occurs before the transaction in question, and
    keeping any state changes they made.

The proof would work as follows:
- Receive data from a peer: Enough information such that they can get a transaction
trace without needing to trust anyone else.
- Perform checks on the data: Hash trees and check that each root matches the expected root.
This roughly includes:
    - Block canonicality
    - State root
    - Transactions root
    - Block canonicality of prior block hashes accessed by EVM

After making those checks, the transactions can be executed using that data.

## The absence of inter-transaction state roots

As discussed, a transaction trace can be obtained trustlessly by starting to trace a whole
block and stopping at the appropriate transaction. IA user is most lik


This leads one to the conclusion that a HemiDemiSemiArchive node must work at the level of
`trace_block` rather than `trace_transaction`.

## Comparison to proofs of EVM execution

Proofs so far discussed are Merkle proofs that show a particular value is part of a tree
with a known root.

There are other sorts of proofs (SNARKs and STARKs) that are used in the Ethereum ecosystem
that can prove a computation has been performed. For example, one could prove that the
state immediately prior to transaction 100 is some value by having a proof system that executes the EVM. Such a proof could be used to provide a quick-to-verify proof for a standalone pre-transaction
state. A node with such a proof could then execute a single transaction to obtain
the trace.

This method involves creating a full ZK-EVM that is aware of all hard fork changes such that
it can provide proofs for all historical transactions. The proof characteristics (cost, size, etc)
would be specific to the particular technique used. This is an active area of research
and engineering in rollup land (L2) and so for clarity is compared to the simpler ready-to-go
model explored here.

## `trace_transaction` from `trace_block`

A node can receive a block and a block header-with-proof (against an accumulator).
They are then able to replay every transaction in that block.

In order to use `trace_transaction` the following can occur:
- Call own node with `trace_transaction`, providing the transaction hash.
- Node accesses an index that maps transaction hashes to block hash and index.
- Node makes a request to a peer with `eth_getBlockByHash` using that block hash to get the
block including transactions.
- Node makes a request to a peer `eth_getArchiveBlockStateProof` using that block hash to
get the state accessed by that block.
- Node checks the proof components, first starting by proving the header, then the state components.
- Node internally starts executing the equivalent of `trace_block`, where any time a state value
is required, the state proof is used (like a small read-write database).

## State proofs as a database

This is an analogy to understand what a proof is and how it can be used. In a Merkle proof
the leaves of the tree are the values. So a proof has two functions:
- Verification: Hash the tree branches until the root is reached and check that the value is correct
with respect to known values (e.g., the state root in the block header).
- Use: Read the leaves of the tree for use in some way.

Hence a proof can be thought of as a small database that stores state values at the start of a
block. It only includes values that are read or written to during that block. Some blocks
will have a lot of values if many different parts of the state are accessed. Some blocks will
re-read or re-write to the same state repetitively, and so will have a tree with fewer values.

So this small database can accompany a block and be used when replaying transactions. If
a transaction modifies state then the small database should be "written to" so that subsequent
transactions have access to the intermediate (intra-block) state. See
[this EF blog post](https://blog.ethereum.org/2015/11/15/merkling-in-ethereum) more on this.


## Proof creation

The data for the archive state proof network could be created by a single archive node
and then distributed to peers. The process would be as follows:

The archive node selects a block and starts executing the transactions. Every time the transaction involves a state lookup it looks up the state and records the value. This would involved (but
not be limited to) applying each opcode (`PUSH1`, `POP`, `MSTORE`, ...) and recording every
`SSTORE`, `SLOAD` and `BLOCKHASH`.

It records the values in a tree structure that matches the tree in the block. Note that
the archive node already has a database containing this data and so this is a second
(restructured) representation of that data. The nature of the existing storage will vary
between nodes and so the most simple method would be to call `trace_block` or equivalent on the node
and parse the result to construct the tree. A more complex approach could be to read
from the archive node database directly.

Once created this tree of state values has a root that is the state root for that block.
The leaves contain any values that were read or written. Additional state accessible by the EVM
such as prior block hashes (up to 256 blocks prior) would be recorded as accessed. These would
then be stored in a structured provable way (E.g., block header with proof against accumulator).
The combined data would then be stored in a format that can readily be used to respond to a
call to `eth_getArchiveBlockStateProof`.


## State proof size estimates

The size for each proof varies and it depends on state accesses.
If blocks can have 30M gas, then the ceiling for proof size of a single block is about 36MB
if every opcode is a state access (`SLOAD`) (source:
[Verkle tree notes](https://notes.ethereum.org/@vbuterin/verkle_tree_eip#Motivation)).
However the normal case is
[estimated](https://blog.ethereum.org/2019/12/30/eth1x-files-state-of-stateless-ethereum)
at closer to 1MB per proof.

This marked variability in proof size is the reason why stateless light clients cannot happen right now. Block producers would need to be gossiped many of these transactions rapidly and 36MB would
be too large. Verkle trees reduce proof sizes to 300B and hence resolve this problem. Verkle
trees do not have as much depth and so require far fewer intermediate nodes in the tree to
be stored than in Merkle Patricia trees.

However, a HemiDemiSemiArchive node does not need to distribute proofs to build blocks. It
does so for old transactions. So all the proofs can be created once and distributed, then
then new proofs added as the chain grows.

A strategy might be to start making proofs when blocks are crated, and aim to have them distributed
within ~1 hour (128 blocks). This allows a node with access to full node data to feed the
archive state proof network before it forgets the state.
As proofs are distributed between nodes, selectively passing around
and storing an anomalous sequence of 36MB blocks is likely to be feasible within an hour.

## Total state proof burden

> Average block state proof is a very significant unknown.

In the worst case the proof for a state-access-heavy block could be 20x larger
than the largest possible block itself (36MB from above vs ~1.8MB, given 16 gas per byte of calldata).
If all blocks were like this, the proofs would take up 600TB (proof-size x blocks = `36e6 * 16e6 = 6e14` bytes).

However the if normal case is closer to 1MB (see above) per proof. Thus the total size
might be closer to 16TB (proof-size x blocks = `1e6 * 16e6 = 16e12`).
This may also be too large, and so testing should be performed for viability of the design.

There may also be additional proofs for state not rooted in the current block:
- The BLOCKHASH opcode accesses the block hash from a preceeding block (up to 256 in the past).
A block header with proof would part of the reponse to `eth_getArchiveBlockStateProof`.

There is no strict upper limit on how big the data can be. With enough nodes the data can
be distributed in very small pieces, so the calculus in part depends on community appetite
to run nodes.

## On the appetite for access

An archive node that stores data as proofs is less efficient than that which can be
achieved by a cleverly designed database on a single machine. This inefficiency
manifests as bandwidth and disk use by users and it is important to decide if
it is worth the benefit?

Access to historical (archive) Ethereum data is very important. It is required in order to:
- Perform basic accounting of anything that is not emitted as an event or log (e.g.,
non-standard token contracts).
- See how contracts call each other in a transaction (was there a delegate call to another contract?)
- Interrogate interesting, novel or aberrant transactions. This category is important for
the community as independent researchers can inspect and decode what has happened. Especially
important for security incidents.

If a transaction is older than one hour, then a normal full node will not be of use for
transaction analysis. The following options are available:
- Run an archive node (hardware cost)
- Rely on charitable and benevolent archive node providers
- Pay someone (an archive node provider)
- Being a customer somehow (wallet user, who is given historical data in exchange for something - trading fees, personal information of value)

Or, as in the case of a HemiDemiSemiArchive node:
- Participating in an inefficient distributed data network (bandwidth cost, perhaps use of
existing hardware).

Realistically, many users will never set up a dedicated machine to run an Archive node and a HemiDemiSemiArchive node as an application on an existing computer could represent the first
time that many users gain access to historical data with:
- No need to trust the accuracy of third party data
- No risk of data censorship
- Better privacy
- No customer-based relationship with unknown or known costs

## Why not IPFS

The `eth_getArchiveBlockStateProof` JSON-RPC method is introduced, why is this needed?.

If we are just storing historical data, why not dump the block state proofs to IPFS and have
people pin subsets of this data? A node could provide JSON-RPC methods (`trace_transaction`)
by first pulling data from any content distribution network (IPFS, bittorrent or Discv5 overlay
network).

The reason for the new method is that a HemiDemiSemiArchive node is likely to have
the full suite of full node JSON-RPC methods are being provided. Hence the node will be
participating in a network and will have infrastructure for sharing data on
a discovery overlay networks. By reusing this network the client can reuse existing code.
This could practically manifest as an overlay network dedicated to archive block state on the
Portal Network.

The use of a JSON-RPC method also allows for cross-client implementation of the method so that
full nodes can serve data into the archive state data network from the chain tip. Without
a unified JSON-RPC method such as `eth_getArchiveBlockStateProof`, nodes may implement
conflicting or client-specific methods that limit interoperability.

## Contrast to an archive node databases

So far we have just considered naive storage of transactions and their proofs. Lets see how
it is done in a client that has all the transactions.

Let's look at Erigon, which is currently at Erigon2. It is interesting to see that the plan
is to move to per-transaction level state history. As such nodes contain all the data, they
can verify the integrity of per-transaction state values. However, those intra-block
intermediate state values cannot be readily shared to peers with an accompanying proof. Hence
this direction is not suitable for a HemiDemiSemiArchive node.

Here is a quote by Alex Sharov in the Erigon discord:
- "Erigon1 - store everything in 1 db file
- Erigon2 - move blocks, recovered senders and getTxByHash index to snapshots
- Erigon3 - move history of state changes (storage/accs/code), indices (logsAddress, logsTopic, traceFrom, traceTo) to snapshots
    - increase state history (and indices) granularity from per-block to per-transaction
    - disable by default storing of receipts/logs, but allow re-generate them by executing only 1  txs (instead of whole block in worst case).
    - when you have historical state snapshots: can faster re-build of latest state (from 0) by not executing txs which do not contribute to latest state (called state ReConstitution).
    - parallel txs execution: to create historical state snapshots
- Erigon4 - move cold parts of lates state (and merkle trie) to snapshots.
"

Additionally, from Alex's [substack](https://erigon.substack.com/p/post-merge-release-of-erigon-dropping):

"History of state will become more granular. Instead of being able to query the state of any account, or contract storage item, or byte code 'as of' certain block height, it will be possible to query these things 'as of' before any transaction in any block."


## Example: Looking for state in a trace

First, using a full node (keeps state for latest 128 blocks) get a block:
```
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "eth_getBlockByNumber", "params": ["finalized", true], "id":1}' http://127.0.0.1:8545 | jq
```
Pick a transaction and call `debug_traceTransaction`, filtering anything that mentions "storage":
```
TX=0x1737d3cb7407b4fbf17e152e2d95cabca691e5fdcc8ae67a48c1a11d4809fe78
```
```
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "debug_traceTransaction", "params": ["'"$TX"'"], "id":1}' http://127.0.0.1:8545 | jq
grep storage -B 30 -A 10
```
Result - there are two SLOADs and one SSTORE. There may be other state accessed, but for now
let's look at these.
```json
[
    {
    "pc": 133,
    "op": "SLOAD",
    "gas": 12063,
    "gasCost": 2100,
    "depth": 1,
    "stack": [
        "0x11",
        "0x27",
        "0x22",
        "0x0",
        "0x9e",
        "0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc"
    ],
    "memory": [
        "0000000000000000000000000000000000000000000000000000000000000000",
        "0000000000000000000000000000000000000000000000000000000000000000",
        "0000000000000000000000000000000000000000000000000000000000000080"
    ],
    "storage": {
        "360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc": "00000000000000000000000017584a148d27ac5d06d87771464dacbaf625ce45"
    }
    },
    {
    "pc": 742,
    "op": "SLOAD",
    "gas": 6922,
    "gasCost": 2100,
    "depth": 2,
    "stack": [
        "0xd0e30db0",
        "0xee",
        "0x0",
        "0xc36e484eaac20fb7a8c6dffae233ec197d51b35adb531c621ac6084f4823412a",
        "0xc36e484eaac20fb7a8c6dffae233ec197d51b35adb531c621ac6084f4823412a"
    ],
    "memory": [
        "00000000000000000000000051ae51ea0cea748c6dce1bebc50de352345fc822",
        "00000000000000000000000000000000000000000000000000000000000000c9",
        "0000000000000000000000000000000000000000000000000000000000000080"
    ],
    "storage": {
        "360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc": "00000000000000000000000017584a148d27ac5d06d87771464dacbaf625ce45",
        "c36e484eaac20fb7a8c6dffae233ec197d51b35adb531c621ac6084f4823412a": "000000000000000000000000000000000000000000000002741eaa348a5547bb"
    }
    },
    {
    "pc": 759,
    "op": "SSTORE",
    "gas": 4730,
    "gasCost": 2900,
    "depth": 2,
    "stack": [
        "0xd0e30db0",
        "0xee",
        "0x16345785d8a00000",
        "0x0",
        "0x28a5301ba62f547bb",
        "0xc36e484eaac20fb7a8c6dffae233ec197d51b35adb531c621ac6084f4823412a"
    ],
    "memory": [
        "00000000000000000000000051ae51ea0cea748c6dce1bebc50de352345fc822",
        "00000000000000000000000000000000000000000000000000000000000000c9",
        "0000000000000000000000000000000000000000000000000000000000000080"
    ],
    "storage": {
        "360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc": "00000000000000000000000017584a148d27ac5d06d87771464dacbaf625ce45",
        "c36e484eaac20fb7a8c6dffae233ec197d51b35adb531c621ac6084f4823412a": "0000000000000000000000000000000000000000000000028a5301ba62f547bb"
    }
    },
]
```
Here we can see `"storage"`, with key-value pairs. These data will
be included in a tree of accessed state. We can already see that
the storage key `3608...2bbc` is repeated. Recall that when building
the tree, we will only record the first value for every accessed key.

That way, we know the state at the start of the block and can update
the record in real time as the transactions are executed.

## Simple transfer exploration: Using `debug_traceTransaction` with `prestateTracer`

The `debug_traceTransaction` method has a `prestateTracer` method.
This returns all state that is accessed (read / write) during the transaction.

Let's pick a transaction that is a simple transfer:
```sh
TX=0x4ebc6a7b170eac0173c6fce706b3be7dc040795140daf2d3ef985ee863a73af5
```
First lets trace it with `callTracer`:
```sh
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "debug_traceTransaction", "params": ["'"$TX"'", {"tracer": "callTracer"}], "id":1}' http://127.0.0.1:8545 | jq
```
Response:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "from": "0x16ad4a024c53d8056b0e38f6df6ca1fb92cb8c11",
    "gas": "0x0",
    "gasUsed": "0x5208",
    "to": "0x7962390d8e104fb6067f0ea88408c6506fbb9967",
    "input": "0x",
    "value": "0xb1a2bc2ec50000",
    "type": "CALL"
  }
}
```
Now with `prestateTracer`:
```sh
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "debug_traceTransaction", "params": ["'"$TX"'", {"tracer": "prestateTracer"}], "id":1}' http://127.0.0.1:8545 | jq
```
Response:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "0x16ad4a024c53d8056b0e38f6df6ca1fb92cb8c11": {
      "balance": "0x19a8149c54fd359",
      "nonce": 500
    },
    "0x1f9090aae28b8a3dceadf281b0f12828e676c326": {
      "balance": "0x5a57d38c712aa53d",
      "nonce": 8908
    },
    "0x7962390d8e104fb6067f0ea88408c6506fbb9967": {
      "balance": "0x0",
      "nonce": 4
    }
  }
}
```
It can be seen that the "from" and "to" accounts have been accessed,
and we have balances and nonces. However, what about `0x1f90...c326`
that has sent 8908 transactions so far?

## Verifying the preStateChange result of a simple transfer
Using the above trace, we can think about whether the returned trace makes sense.
Why do we have three parties in a simple transfer from A to B?

Let's use the `preStateTracer` with `diffMode` on:
```sh
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "debug_traceTransaction", "params": ["'"$TX"'", {"tracer": "prestateTracer", "tracerConfig": {"diffMode": true}}], "id":1}' http://127.0.0.1:8545 | jq
```
Response:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "post": {
      "0x16ad4a024c53d8056b0e38f6df6ca1fb92cb8c11": {
        "balance": "0xe29cde97b99bc1",
        "nonce": 501
      },
      "0x1f9090aae28b8a3dceadf281b0f12828e676c326": {
        "balance": "0x5a57d42f0ad715c5"
      },
      "0x7962390d8e104fb6067f0ea88408c6506fbb9967": {
        "balance": "0xb1a2bc2ec50000"
      }
    },
    "pre": {
      "0x16ad4a024c53d8056b0e38f6df6ca1fb92cb8c11": {
        "balance": "0x19a8149c54fd359",
        "nonce": 500
      },
      "0x1f9090aae28b8a3dceadf281b0f12828e676c326": {
        "balance": "0x5a57d38c712aa53d",
        "nonce": 8908
      },
      "0x7962390d8e104fb6067f0ea88408c6506fbb9967": {
        "balance": "0x0",
        "nonce": 4
      }
    }
  }
}
```
So this third address gained 698362917000 wei (698 Gwei). It will be the tip payment to the
block producer (miner). Let's check by seeing what the priority fee per gas was.

```sh
$ curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "eth_getTransactionByHash", "params": ["'"$TX"'"], "id":1}' http://127.0.0.1:8545 | jq
```
Returns:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "blockHash": "0x67dd0d35b6572c6d7443ef1124c4b7bafd962867958106be273531493257ce4a",
    "blockNumber": "0xff81ab",
    "from": "0x16ad4a024c53d8056b0e38f6df6ca1fb92cb8c11",
    "gas": "0x5208",
    "gasPrice": "0x1386791c33",
    "maxPriorityFeePerGas": "0x1fb6fd1",
    "maxFeePerGas": "0x1825052d1b",
    "hash": "0x4ebc6a7b170eac0173c6fce706b3be7dc040795140daf2d3ef985ee863a73af5",
    "input": "0x",
    "nonce": "0x1f4",
    "to": "0x7962390d8e104fb6067f0ea88408c6506fbb9967",
    "transactionIndex": "0xf6",
    "value": "0xb1a2bc2ec50000",
    "type": "0x2",
    "accessList": [],
    "chainId": "0x1",
    "v": "0x0",
    "r": "0xf7015cd91089482ea032eee5b3ce0451946cd869793cec8b72da7a33a0f40124",
    "s": "0x31009337e54b43ce8e48b967c6c9b0c49671c21d19cc806542895bca36244a61"
  }
}
```
So the max the user was prepared to give the miner was:
max_priority_fee_per_gas was 0x1fb6fd1 = 33255377 wei/gas = 0.33 Gwei/gas.
```
max priority fee (wei) = max priority fee per gas (wei/gas) * gas used (gas)
max priority fee = 0x1fb6fd1 * 0x5208
max priority fee = 33255377 * 21000
max priority fee = 698362917000 = 698 Gwei
```
This matches what we observed being changed in the trace earlier.

As the full priority fee was paid, we can assume that the base fee was below
the users max fee.
```sh
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "eth_getBlockByHash", "params": ["0xff81ab", false], "id":1}' http://127.0.0.1:8545 | jq
```
Returns the block, including the basefee: 0x13847dac62 (wei/gas).

We know that the priority fee will be charged fully if the base fee plus the priority fee
are within the max:
```
max fee per gas (wei/gas) > base fee per gas (wei/gas) + priority fee (wei/gas)
0x1825052d1b > 0x13847dac62 + 0x1fb6fd1
103700311323 > 83827207266 + 33255377
103 Gwei/gas > 83.8 Gwei/gas + 0.33 Gwei/gas
103 Gwei/gas > 84.1 Gwei/gas (true)
```
The condition holds. If this was not true, then only part of the priority fee would be given and we would have expected a different state change.

This confirms that the third address and state change are the protocol transaction fee.

## Basic contract interaction exploration: Using `debug_traceTransaction` with `prestateTracer`

Let's pick a transaction that is a contract interaction:
```sh
TX=0x161b0f272bd99727861d7383e5cf239e6f08074697b1a7e5771246e0fc50e071
```
First lets trace it with `callTracer`:
```sh
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "debug_traceTransaction", "params": ["'"$TX"'", {"tracer": "callTracer"}], "id":1}' http://127.0.0.1:8545 | jq
```
Response:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "from": "0x58803db3cc22e8b1562c332494da49cacd94c6ab",
    "gas": "0x1166b",
    "gasUsed": "0x108ce",
    "to": "0xae7ab96520de3a18e5e111b5eaab095312d7fe84",
    "input": "0x095ea7b3000000000000000000000000000000000022d473030f116ddee9f6b43ac78ba3ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
    "output": "0x0000000000000000000000000000000000000000000000000000000000000001",
    "calls": [
      {
        "from": "0xae7ab96520de3a18e5e111b5eaab095312d7fe84",
        "gas": "0xf572",
        "gasUsed": "0x2047",
        "to": "0xb8ffc3cd6e7cf5a098a1c92f48009765b24088dc",
        "input": "0xbe00bbd8f1f3eb40f5bc1ad1344716ced8b8a0431d840b5783aea1fd01786bc26f35ac0f3ca7c3e38968823ccb4c78ea688df41356f182ae1d159e4ee608d30d68cef320",
        "output": "0x00000000000000000000000047ebab13b806773ec2a2d16873e2df770d130b50",
        "calls": [
          {
            "from": "0xb8ffc3cd6e7cf5a098a1c92f48009765b24088dc",
            "gas": "0xb9bc",
            "gasUsed": "0xb04",
            "to": "0x2b33cf282f867a7ff693a66e11b0fcc5552e4425",
            "input": "0xbe00bbd8f1f3eb40f5bc1ad1344716ced8b8a0431d840b5783aea1fd01786bc26f35ac0f3ca7c3e38968823ccb4c78ea688df41356f182ae1d159e4ee608d30d68cef320",
            "output": "0x00000000000000000000000047ebab13b806773ec2a2d16873e2df770d130b50",
            "type": "DELEGATECALL"
          }
        ],
        "value": "0x0",
        "type": "CALL"
      },
      {
        "from": "0xae7ab96520de3a18e5e111b5eaab095312d7fe84",
        "gas": "0xa632",
        "gasUsed": "0x698c",
        "to": "0x47ebab13b806773ec2a2d16873e2df770d130b50",
        "input": "0x095ea7b3000000000000000000000000000000000022d473030f116ddee9f6b43ac78ba3ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
        "output": "0x0000000000000000000000000000000000000000000000000000000000000001",
        "type": "DELEGATECALL"
      }
    ],
    "value": "0x0",
    "type": "CALL"
  }
}
```
This `callTracer` configuration of the `debug_TraceTransaction` method provides information
that maps to how developers think about contracts being protocols. The method exploses how
the transaction sequentially interacts with different protocols. E.g., First in protocol a,
then visiting protocol b, then protocol c.


Now with `prestateTracer`:
```sh
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "debug_traceTransaction", "params": ["'"$TX"'", {"tracer": "prestateTracer"}], "id":1}' http://127.0.0.1:8545 | jq
```
Response:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "0x2b33cf282f867a7ff693a66e11b0fcc5552e4425": {
      "balance": "0x0",
      "code": "608060/* Snip (entire contract bytecode) */0b0029",
      "nonce": 1
    },
    "0x47ebab13b806773ec2a2d16873e2df770d130b50": {
      "balance": "0x0",
      "code": "0x608060/* Snip (entire contract bytecode) */a90029",
      "nonce": 1
    },
    "0x58803db3cc22e8b1562c332494da49cacd94c6ab": {
      "balance": "0x13befe42b38a40",
      "nonce": 54
    },
    "0xae7ab96520de3a18e5e111b5eaab095312d7fe84": {
      "balance": "0x4558214a60e751c3a",
      "code": "0x608060/* Snip (entire contract bytecode) */410029",
      "nonce": 1,
      "storage": {
        "0x1b6078aebb015f6e4f96e70b5cfaec7393b4f2cdf5b66fb81b586e48bf1f4a26": "0x0000000000000000000000000000000000000000000000000000000000000000",
        "0x4172f0f7d2289153072b0a6ca36959e0cbe2efc3afe50fc81636caa96338137b": "0x000000000000000000000000b8ffc3cd6e7cf5a098a1c92f48009765b24088dc",
        "0x644132c4ddd5bb6f0655d5fe2870dcec7870e6be4758890f366b83441f9fdece": "0x0000000000000000000000000000000000000000000000000000000000000001",
        "0xd625496217aa6a3453eecb9c3489dc5a53e6c67b444329ea2b2cbc9ff547639b": "0x3ca7c3e38968823ccb4c78ea688df41356f182ae1d159e4ee608d30d68cef320"
      }
    },
    "0xb8ffc3cd6e7cf5a098a1c92f48009765b24088dc": {
      "balance": "0x0",
      "code": "0x608060/* Snip (entire contract bytecode) */cd40029",
      "nonce": 10,
      "storage": {
        "0x54b2b2de1ae6731a04bdbca30cee71852851cfcd3298aaf29f4ebff9452b27ad": "0x00000000000000000000000047ebab13b806773ec2a2d16873e2df770d130b50",
        "0x8e2ed18767e9c33b25344c240cdf92034fae56be99e2c07f3d9946d949ffede4": "0x0000000000000000000000002b33cf282f867a7ff693a66e11b0fcc5552e4425"
      }
    },
    "0xdafea492d9c6733ae3d56b7ed1adb60692c98bc5": {
      "balance": "0x106001246036090a",
      "nonce": 251257
    }
  }
}
```
Note that contract code is included in the response. Contract code present at the time of
the transaction is important because while many contracts do not change, some may.
For example, through the SELFDESTRUCT opcode or CREATE2 opcode a contract code may change
in the transaction prior.

The contract code has been snipped out and the total size of the pre-snipped response
was 78KB. Hence for complex contract calls this can amount to a significant cost.

Let's compare the result to the `diffMode`
```sh
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "debug_traceTransaction", "params": ["'"$TX"'", {"tracer": "prestateTracer", "tracerConfig": {"diffMode": true}}], "id":1}' http://127.0.0.1:8545 | jq
```
Response:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "post": {
      "0x58803db3cc22e8b1562c332494da49cacd94c6ab": {
        "balance": "0xeb6c743e9ca94",
        "nonce": 55
      },
      "0xae7ab96520de3a18e5e111b5eaab095312d7fe84": {
        "storage": {
          "0x1b6078aebb015f6e4f96e70b5cfaec7393b4f2cdf5b66fb81b586e48bf1f4a26": "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff"
        }
      },
      "0xdafea492d9c6733ae3d56b7ed1adb60692c98bc5": {
        "balance": "0x10600622a2b46b7a"
      }
    },
    "pre": {
      "0x58803db3cc22e8b1562c332494da49cacd94c6ab": {
        "balance": "0x13befe42b38a40",
        "nonce": 54
      },
      "0xae7ab96520de3a18e5e111b5eaab095312d7fe84": {
        "balance": "0x4558214a60e751c3a",
        "code": "0x608060/* Snip (entire contract bytecode) */410029",
        "nonce": 1
      },
      "0xdafea492d9c6733ae3d56b7ed1adb60692c98bc5": {
        "balance": "0x106001246036090a",
        "nonce": 251257
      }
    }
  }
}
```
We only have three changes:
- `0x5880...c6ab`
    - Nonce increase with a balance decrease by 0.0014 ether
    - Likely sender paying tx fee
- `0xae7a...7fe84`
    - Contract with storage value `0x1b60...4a26` changed to `0xffff...ffff`.
- `0xdafe...98bc5`
    - Large nonce account with small balance increse
    - Likely the miner receiving tx fee

## Summary of `debug_traceTransaction` explorations

The `debug_traceTransaction` is a convenient method to get information required to
produce an archive block state proof. It has different tracers available.

The `callTracer` tracer is good for accessing all the opcodes, but may not be necessary
due to the existence of other configuration options.

The `prestateTracer` tracer is good for identifying every part of state read/written
by a transaction. This is good for **creating a proof** because you can enumerate transactions
for a whole block and store every part of state needed for that block.

The `prestateTracer` tracer with `diffMode` configuration shows what state values changed during
a block. This is good for when you are **using a proof** to replay a specific transaction.
Preceding transactions are enumerated and the true state immediately prior to the transaction
can be known.

## Modifications to `prestateTracer` for proofs

The `prestateTracer` tracer could be useful because clients have them implemented already.
They can minimally modified to be useful in the generation and usage of archive block state proofs.
These changes are summarised as follows:

- Proof generation
    - Modify `prestateTracer` tracer to output a state proof
    - The tracer produces only the state required for a block
    - All transactions in a block are processed and the result aggregated
    - Duplicate state access is ignored, only store the first value encountered for a given key
    - Output is used for an `eth_getArchiveBlockStateProof`
- Proof consumption
    - Modify `prestateTracer` tracer with `diffMode` on to recreate pre-transaction state.
    - This is for executing an `debug_traceTransaction` for a specific transaction.
    - The tracer is used to find the state immediately prior to the transaction in question.
    - The tracer uses a block state proof, then applies a transaction trace with `diffMode`
    to quickly see what is different after each transaction
    - The effect of all preceeding transactions are applied and then the transaction of interest
    can be traced completely
    - The trace of the transaction-of-interest can then be executed with whatever trace is desired
    such as `callState`.

## On contract bytecode duplication

A node responsible for providing`trace_block` must have the contract code for every contract
involved in that block. It realistically needs to have this code and cannot just keep the
code hash and point to another overlay network because this would be a DoS burden for the
unfortunate nodes who happen to be responsible for popular contracts.

Nodes are responsible for many non-contiguous blocks. Functionally immutable contracts
that are popular will probably re-appear in these disparate blocks. Hence, a node can store
contract code in a way that doesn't duplicate this data. Thus, while it may seem that the problem
of duplication of contract bytecode amplifies the total size of the network-wide database,
the true size is less.

The per-node archive state database size should therefore not be calculated by naive estimates.
It will depend on the node radius, and the number and frequency of "common contracts". With
Contract code duplication will decrease if radii are larger and number and frequency of common
contracts is higher.

## debug_traceTransaction for slots, eth_getProof for slot proofs

Recall that the state tree is nested: Each account has a storage root for a storage tree.
This storage root is not returned as part of `debug_traceTransaction`. To
construct the leaf data of the state root tree, the Recursive Length Prefix (RLP) encoded
account data is all required. So to provide someone with a specific storage value (e.g., some storage slot that will be accessed during a block) in the storage tree, the RLP data must be reconstructed, hashed and proved in the first tree. This requires that the code hash, nonce and balance for every account accessed must be part of the proof data.

A retrospective look at one block can reveal all the leaf data that is needed to execute that block. Aggregation of all those values into one big tree is the proof. Imagine that a block only accessed one storage value from one contract (AKA account). Here is the data that would be in the proof:

- Storage key
- Storage value
- Account storage root (of storage tree, using key/value)
- Account code hash
- Account nonce
- Account balance

A call to debug_traceTransaction with the prestateTracer may return balance, code, nonce and storage, and some fields may absent.

If only the balance of an address is accessed (there is code etc that is not accessed), the other fields are still required. Once obtained, the other fields are RLP encoded to get the account leaf, then the hash of that encoded data is the account node.

Thus the prestateTracer is necessary (to know which storage keys are accessed) but insufficient (does not get account storage root or other unaccessed account state fields).

A convenient approach would be to call `eth_getProof` for every account.
This will provide the account node (account state root), the account value
(RLP encoded data) and the storage proof against the storage root in that encoded data.

Thus we have a new step: `eth_getProof`:
1. eth_getBlockByNumber
2. For each transaction, get novel state with `debug_traceTransaction` with prestate tracer
3. Combine all required storage slots for all accounts accessed
4. Request a single proof for each account with `eth_getProof`
5. Combine all account proofs in a tree to form the state proof.

## State

## Validation

In order to test the viability of `eth_getArchiveBlockStateProof`, the following steps could
be followed:

- Proof construction
    - Get a recent block using a full node and execute transactions as traces
    (`debug_traceBlock`, `trace_block`)
    - Record every state access in a tree
    - Store the tree as a proof for state access for that block
    - That tree would be the thing returned by the node in response to the
    `eth_getArchiveBlockStateProof` method.
- Proof verification
    - Compute the hashes of the state tree and any other proofs involved.
    - Check the values match those expected (e.g., in the block header)
- Proof use
    - Call `ethGetBlockByHash`, (pretendend that the trace is not available) and get transactions.
    - Modify an EVM implementation so that when state access is required it uses the newly created
    tree. This could be simulated as a modified `trace_block` style function.
    - Produce the trace block output and compare to the the original `trace_block` output.
- Proof burden estimate
    - Compute the proofs for a large sample of blocks and estimate the total proof burden.

## Integration

If validation shows that the direction is viable, then subsequent steps could be:
- Explore if people are interested in the method and HemiDemiSemi Archve nodes more generally.
- Write a specification for the archive block state sub-protocol in the Portal Network
    - Consider what messages and data types would be required.
- Explore security risks and other challenges
    - Data limits and 48MB messages in uTP/Discovery
    - DoS attacks on Portal Nodes
    - Foresee issues encountered on the switch to Verkle trees.
- Implement the network in Portal Network clients
- Create all the proofs using an archive node
- Distribute to peers
