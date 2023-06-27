---
layout: page
title:  "Sharded historical EVM tracing"
permalink: /ethereum/protocol/tracing
toc: true
---

In the last post, we looked at how you could use the state root in a block header to
check a database of values that are required for replaying an Ethereum block.

Now let's step back and look at the benefit this provides.

## A money computer with consensus?
Introducing a newcomer to Ethereum goes a bit like this:
```
Superintendent Chalmers: So you are telling me that I can push money
into programs and that strangers can agree on every little step in
each program?

Skinner: Yes!

Superintendent Chalmers (shaped like a 2TB NVMe): May I see it?

Skinner: ... No
```
You can see all the programs! Except when you can't.

  - [A money computer with consensus?](#a-money-computer-with-consensus)
    - [Simply inspecting the programs](#simply-inspecting-the-programs)
    - [Trust issues](#trust-issues)
    - [Tracing Recipe](#tracing-recipe)
    - [Expansion](#expansion)
    - [Parcel making](#parcel-making)
    - [Parcel taking](#parcel-taking)
    - [Use cases](#use-cases)
    - [Entry points](#entry-points)
    - [Presentation](#presentation)
  - [Structural overview](#structural-overview)
    - [Elements](#elements)
    - [Canonicality](#canonicality)
    - [Verkle tries](#verkle-tries)
  - [Halfway measures](#halfway-measures)

This is an exploration to accompany a prototype ([https://github.com/perama-v/archors](https://github.com/perama-v/archors)) of sharded archive node architecture.

### Simply inspecting the programs

After all, the EVM is a machine that executes step by step. We can look
at those steps or we can look a executive summaries of the program via
events.

Events are good, but sometimes something happens in the EVM that is
interesting but not emitted as an event.

A globally consistent money program should be inspectable by all! It is,
of course, but you need an archive node. Those are larger (~2TB) than
your average computer and so your average computer user cannot
inspect these money programs (transactions).

### Trust issues

Money programs are sensitive things. It's okay to ask a stranger's
website for a summary of an old money program most of the time. Yet
there may be situations where this is not ok. It's hard to enumerate
them but the idea is this:

1. IF Ethereum is used for very important things,
2. THEN it will be very important for users to be very sure about the things that it does.

It's critical to build 2 so that 1 is enabled.

So, what we are talking about here is using Ethereum for something
that is important, but not so important that you have set up a dedicated
2TB machine for your purpose.

Some categories:
- Censorship of "unsavoury" historical programs.
- Edits to "sensitive" historical programs.
- Logging of people interested in "special" historical programs.

The quoted terms above refer to situations where some powerful authority
seeks to control an individual or group. Something as simple but harmful
as "prevent this minority group that is using Ethereum from seeing the truth
about the situation". A data provider could selectively
control/modify/track accesses to a specific contract at the request of
a third party.

Is it likely? No, merely possible.

### Tracing Recipe

Old Transaction Inspection

Ingredients:
- Block with transactions
- State for EVM
- Blockhashes for EVM
- Programs (contract code)

Steps:
1. Warm up EVM by inserting state, blockhashes and programs
2. Add a transaction and run the program it describes
3. Record the program
4. Repeat 2-3 for all transactions
5. Present the programs in a way that is useful for the context

This recipe only requires ~10MB of data but produces about ~1GB of output.
So its best to store the ingredients and bake as required.

### Expansion

As indicated about the transaction traces are much larger than the programs.
So the best way to disseminate and store them is to provide the ingredients
and and "trace on demand".

Luckily, this is also the best way to do this trustlessly.
A block is essentially a verifiable list of how to start a handful of
programs. The programs themselves are verifiable (code hash) and use values
that are also verifiable.

Hence, we can pass around small parcels that deterministically and trustlessly
can be unfolded into large and comprehensive records.

### Parcel making

Archors ([https://github.com/perama-v/archors](https://github.com/perama-v/archors)) is a prototype that constructs these parcels. It can also unwrap the
parcels for use to produce the comprehensive whole-block trace without
sacrificing trust.

One can make these parcels available to others in a torrent or
bounce them around in a P2P network like Portal ([https://github.com/ethereum/portal-network-specs](https://github.com/ethereum/portal-network-specs)),
or host them as a public good bootstrap.

Parcels are made using archive nodes. Once made, the archive node is
not needed. Parcels can also be made near the chain tip using non-archive nodes.

### Parcel taking

A recipient of a parcel does not need to trust anyone. They only
need to have access to a mechanism to verify block hash canonicality.

They can reject bad parcels.

### Use cases

By replaying an old block using a parcel one can see things
that full (non-archive) nodes cannot:

- Changes to stored values
- Program flow between different functions
- Program flow between different contracts
- Values sent to and returned by functions

This is a large design space and there are many interesting areas
to explore.

Importantly a transaction trace has "everything that happened",
but for a user, this might be insufficient for consumption.
It is but the first step in local transaction inspection.
A parser will also benefit from:
- Function names (via [https://4byte.directory](https://4byte.directory))
- Function argument names (via source code verification from [https://sourcify.dev](https://sourcify.dev))
- Address names and tags (no known robust source, but an example here: [https://github.com/perama-v/min-know/tree/main/data/samples/todd_nametags/raw_source_nametags](https://github.com/perama-v/min-know/tree/main/data/samples/todd_nametags/raw_source_nametags)

### Entry points

Additionally, some thought can be given to the question "which transactions
would you like to trace?".

For forensic analysis, one might already know of a specific transaction
from out-of-band channels.

For a user introspecting on a single address, the Unchained Index
[https://trueblocks.io/data-model/the-index/](https://trueblocks.io/data-model/the-index/)
can be used to discover which transactions are of interest.
The use of bloom filters to allows a user to fetch a subset of the appearance index
relevant for a particular address.

### Presentation

Some nice ways single-block traces could be presented might be to use that data as
a local backend for one of:
- EthTx (internals [https://github.com/EthTx/ethtx](https://github.com/EthTx/ethtx), frontend [https://github.com/EthTx/ethtx_ce](https://github.com/EthTx/ethtx_ce))
- Otterscan [https://github.com/otterscan/otterscan](https://github.com/otterscan/otterscan)


## Structural overview

The recipe analogy is ok, but more details are provided here.

One can construct a static ~30TB database that consists of proofs of historical state data.
That database is divisible at the block level. So, if you as a user were only interested
in the things that your address has been involved in, you could:

1. Pull the Unchained Index [https://trueblocks.io/data-model/the-index/](https://trueblocks.io/data-model/the-index/) from IPFS
2. Find which blocks your address appeared in
3. Obtain the proofs for historical state only for those blocks
4. Get the blocks (eth_getBlockByNumber)
5. Use an EVM like [https://github.com/bluealloy/revm/tree/main](https://github.com/bluealloy/revm/tree/main) and load it with the state for one block, then trace the block
6. Parse the trace however you please
7. Pull some other rich data (4 byte signatures / sourcify function arguments / nametags) for
a human readable context

The total burden for you as a single user is low because you only need a fraction of the static
database. So while 30TB is seemingly a too large, in practice it is a distributed load.

### Elements

The database consists of state that the EVM needs. At the chain tip a transaction may access
any part of state (any address, any storage slot). However, once the block has been finalized
we know which part it actually accessed.

So, for each block we look at the block, see what was needed and then create a discrete set of
proofs for those exact values.

This covers:
- Accounts
- Storage

The EVM also has the option to access to the previous 256 blockhashes. In retrospect, we can
see which were accessed and also include these values in the bundle of state data.

This covers:
- Accounts
- Storage
- Blockhashes

### Canonicality

A user can verify the proofs as long as they have access to a mechanism to check if
a blockhash is part of the canonical set of blockhashes. Such a mechanism is
provided by Portal
([https://github.com/ethereum/portal-network-specs](https://github.com/ethereum/portal-network-specs)).

### Verkle tries

Proofs are rooted in the state roof in the block header. If Ethereum deploys `EIP-6800 Ethereum state using a unified verkle tree`
[https://github.com/ethereum/EIPs/pull/6800](https://github.com/ethereum/EIPs/pull/6800)
then new headers will have verkle proofs. So state proof bundles created for those blocks
will have verkle proofs too. These proofs would be smaller than their merkle proof equivalents.

However every block prior to that fork will still have a merkle trie state root in the header.
So bundles consisting of merkle proofs for historical state would be required.

Hence one might envision a static database that consists of bundles with merkle trie innards
up to some block, then verkle trie innards after that block.

## Halfway measures

The overall idea is to enable someone to pull <10MB of data and then
trustlessly generate an EVM trace that could be ~1GB.

At present infrastructure providers serve debug_traceBlock endpoints which involves
two problems:
- Trusting a counterparty (for data integrity, etc)
- Receiving 1GB over the network

It could be feasible for these providers to send these parcels and say "trace it yourself".
The user receives the parcel (<10MB), checks the proofs and runs an EVM locally. It expands
to the ~1GB trace client side.

In this way an infrastructure provider could improve trust with users and also be poised
to bootstrap a distributed network of these 30TB of parcels.
