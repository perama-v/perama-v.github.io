---
layout: page
title:  "Protocol poking"
permalink: /ethereum/protocol/poking/part_6
toc: false
---


# Part VI - Acquisitions


- Hypothesis: Remote 4byte and Sourcify databases can be distributed by generalising the
principles used in the Unchained Index and address-appaearance-index.
- Methods: Create minimum database architecture and publishing requirements that can be followed
to transform a centralised database in to a decentralised one.
- Conclusion: The TODD and GAMB ERCs provide a basic procedure that plausibly are a pathway
for 4byte, Sourcify and other databases to be distributed.

(Continued from [poking part 5](part_5.md))

## Some Hunting

Now that the address-appearance-index is conveying useful information for a user,
let us turn examine what it means to be calling an API for the log signature
decoding and for fetching the contract abi.

So I set out to obtain the 4byte directory and the Sourcify repository.

Where is the starting point to get "the data".

In the case of the 4byte registry there is an api at 4byte.directory and
a repository on github. Additionally, there are hints that people are scraping
and tucking away the index themselves in the dark. There is a sense that
the registry seeks to be a hungry black hole that absorbs all the mappings it can.
This is great, and the site is undoubtedly a critical public good. Yet there is
also opacity as to how to enthusiastically interact with the directory. If you wanted
to help out, what should you do? Certainly use their site to add mappings. How about
contributing to redundancy and availability.
If you scrape the directory to host for others, you quickly
fall out of sync as it hoovers more mappings. The directory seems to be posted to github
semi frequently as a transparent backup, but is a little unclear if this is the full collection.
There also exist other websites that use 4byte mappings, but in opaque and closed ways.

As a user with a bit of hard drive space, it is hard to help out. I feel that we can go
far if we have software that "helps by default, unless you tweak the settings".

In the case of Sourcify, it is great that the Solidity compiler automatically bakes metadata
on chain. The idea here is that if you once make your source code available, it will be scraped and
absorbed into the Sourcify repository. So the repository grows continuously and Sourcify
provide an API to consume and contribute to. If you want to help out further, the
repository is published via IPNS, which means that as the root hash of the repository changes,
one can locate the data. Now this is okay, but what if Sourcify goes away? Is there a clear
path to keep the system growing? IPNS feels a little brittle in this regard.

## Some MetadataTrials

So I have been poking around inside old transactions using the address-appearance-index
and a "simulated portal node". I have the transaction event logs and their
associated contract addresses. I `eth_GetCode` and snip off the metadata. De-CBOR-it, then
obtain the IPFS CID. Then I am trying these hashes in different places to fetch the metadata.

My local ipfs node stares-blank faced with each `ipfs get <cid>`. Ok, maybe my node is
not connected to the right regions of the IPFS graph. So I start trying other public
gateways and it is a similar story. Timeouts and nothing. Now some contracts are obscure,
but after stalling at fetching contracts like `WETH`, it looks like there are not many
Sourcify Home Enthusiasts. With the exception of William Mitsuda (of [Otterscan](#otterscan)
fame), who goes above and beyond once more
https://github.com/wmitsuda/otterscan/blob/develop/docs/ipfs.md#pinning-sourcify-locally.

Yet I cannot get find the data. So that makes me think. At no point, might I add, am
I trying to criticise the approach of anyone involved in [4byte](#4byte-directory) and
[Sourcify](#sourcify), whom I hold in deep respect.

Are there simple hacks to make these systems hum and whir?
https://github.com/ethereum/sourcify/issues I can see the teams behind these goods
wrestle with the right way to tweak things.

## Some Ideals

TrueBlocks UnchainedIndex may be sourced through such a hosted gateway
that is baked into trueblocks-core. It "just works" when you run `chifra init`, but
you can also scrape and pin-at-home to help. This is a good model, because
it means that in a pinch, TrueBlocks proper could turn off their computers and
there is a probability that the index will stay alive (perhaps at slower download
speeds).

Really, this is a good model because in reality, a network does not instantly materialise.
Every network starts with one node and embarks on a journey. By having a strong and
reliable start, the network can gain momentum. When the network becomes critical
infrastructure for someone, they will then be incentivised to participate.

Idealistic transitions:
```
Begin -> Provide supports -> Gather community -> Reduce supports -> Community adds support
```
Realistic transitions:
```
Begin -> Gather community -> Community finds concept unreliable -> Moves on
```
## Some Criteria

In designing the derivative index ([address-appearance-index](#address-appearance-index-specs)),
based on the [Unchained Index](#unchained-index)),
I had to wrestle with two ideas:

1. The index is never complete
2. One should be able to get a useful, small part of the index

Regarding 1. That the index is never complete messes with root hashes. The solution here is
showcased by
the UnchainedIndex. You seal up pieces and tack them onto old ones. The index becomes a
blockchain of sorts itself. This is not a new idea, Ligi
musing about a PoA-style mechanism to grow the Sourcify database in such a way
https://github.com/ethereum/sourcify/issues/346.

I feel that the solution is to make a decision never to touch old parts of the index.
How exactly that is determined is not super criticial. An arbitrary criterion may be used,
and as long as it conforms to some published spec it doesn't matter.

So what do I mean? As Jay (of TrueBlocks fame) puts it:

```
Unchained-index-like structure: a web 2.0 database broken up in time-ordered dependent chunks
so as to facilitate distribution on content-addressable storage mediums.
```

- Web 2.0 database (An unwieldly evolving blob on a server)
- Broken up (a schema that may be followed to divide a databese)
- Time-ordered (when new information comes along, we add it)
- Dependent (we add to old bits information)
- Chunks (a manageable piece of data)
- Facilitate (software that lets you help by default)
- Distribution (peer to peer sharing for availability)
- Content-addressable (The data never changes, so you can fetch it by hash)
- Storage mediums (Software that keeps data organised)

Which is just such a lovely way of putting it.

Regarding 2 (One should be able to get a useful, small part of the index).
This is a criterion I think is important if you want to get distribution.
You should be able to "chip in as a small fry". If 1000 people each contribute 10GB of
spare space, you end up with a robust distributed network.

The trick is working out how to convince people that they should be downloading and
serving that 10GB. You have to have a reason and it has to be slick. Torrenting works because
you want some file to consume. Not because you believe in the everlasting preservation of
that file. They accidentally seed a file for a week before they know what's happening.

Index data needs accidental seeders.

So, what carrot can we offer to people so that they accidentally help out the 4byte directory
or the Sourcify registry? Earlier I said that the criterion used to partition the
data may be arbitrary.

This is partially true, there needs to be careful consideration about the plight of the end user.
Let's examine the plights for each database.

- [Unchained Index](#unchained-index)
    - I have a full node and an address. I want to trace the transactions.
    - The bloom filter informs me that I NEED chunks x, y and z.
- [address-appearance-index](#address-appearance-index-specs)
    - I have a portal node and an address `0xabcd...1234`. I want to look at my
        previous transactions.
    - I NEED the chapter of the index that my address is part of (`chapter_0xab`).
- [4byte directory](#4byte-directory)
    - I have a function method or a log topic signature that I can't interpret (`0xffb5b6af`).
    - ??? I NEED ... The chapter of the registry the my signature is part of (`chapter_0xff`).
- [Sourcify registry](#sourcify)
    - I have a contract with identifier  and I don't know how to interact with it. Identifier may be:
        - Deployed address (`0xabcd...1234`)
        - `eth_getCode` metadata IPFS CID if present (`Qm1234...wxyz` ), or swarm hash.
        - Hash of the runtime bytecode (as many contracts are the same)
    - ??? I NEED ... The chapter of the registry that contains contract identifiers starting
    with (`0xab` or `Qm12...`).

## Some TimeOrdering

So how do we reconcile the different needs:

- Immutable chunks
- Functional chapters

Simply!

Each published chunk contains the functional chapters.

- chunk x
    - Useful chapter <- Alice downloads this
    - Other chapters
- chunk x + 1
    - Useful chapter <- Alice downloads this
    - Other chapters

So every time a new chunk comes out, you get the part you are interested in. However, the
terminology is vague. What are chunks, chapters, subsets and shards?.
Jay had the idea to copy the publishing industry. Let's try that.

If volumes are time based and chapters are targeted to user desires:

- volume x
    - Useful chapter <- Alice downloads this
    - Other chapters
- volume x + 1
    - Useful chapter <- Alice downloads this
    - Other chapters

That seems clearer. A user wanting to decode signature `0xffb5b6af` will need:

- volume 1, chapter `0xff`
- volume 2, chapter `0xff`
- ...
- volume 36, chapter `0xff`

## Some Bookish

So where does that leave us? We define time-based volumes and desire-based chapters that
make sense for each index/database.

- Unchained Index
    - A volume: Data covering a range of blocks (`015508866-015511829.bin`) that contain approximately `2_000_000` address appearances (~25MB).
    - A chapter: No such concept. You decide which volumes you want by using a bloom
    filter against your address.
- address-appearance-index
    - A volume: Data covering next 100_000 blocks in sequence (15_000_000 to 15_099_999).
    - A chapter: Addresses that share two starting characters (`0xab...`)
- 4byte registry
    - A volume: Data covering next (?) quarter in sequence (2022 Q1).
        - Could also be size based. e.g., new volume every xMB.
    - A chapter: Signatures that share two starting characters (`0xff...`)
- Sourcify registry
    - A volume: Data covering next (?) quarter in sequence (2022 Q1)
        - Could also be size based. e.g., new volume every xMB.
    - A chapter: Contract metadata identifiers that share starting characters (`Qm12...`)

The paramater design space looks like this:

- Volume definition defines the shortest time period in which someone can add new immutable pieces
of data.
    - One can define short time periods (daily)
        - If the definition is too small, the pieces of data might be tiny, creating
        a hassle to download or store (<10KB is likely too small)
    - One can define long time periods (yearly)
        - If the definition is too large, the data cannot be refreshed quickly
        (address-appearance-index data that lags by one year is likley too stale)
    - Publishing cadence is flexible either way:
        - One can publish at a slow cadence, pushing multiple volumes at once.
        - One can push at the max cadence, keeping the network plump with data.
    - Question to ask: "How long before a user will become frustrated that the latest data is not
    available?". My gut feeling is:
        - UnchainedIndex: 12 hours (users are data nerds)
        - address-appearance-index: 2 weeks (users are regular people who forget old transactions)
        - 4byte registry: 4 months (users want "a pretty comprehensive" set of mappings)
            - Maybe not upset if recent signatures haven't appeared yet.
        - Sourcify registry: 4 months (users want "a pretty comprehensive" set of contracts)
            - Maybe not upset if deployed-last-month hot contracts haven't appeared yet.
- Chapter definition defines the smallest subset of the index/database that
a user will likely obtain.
    - One can define narrower subsets (`0xffff...`)
        - If the definition is too small, the user contributes less to the network because
        there is less overlap with other user desires.
    - One can define broader subsets (`0xf..`)
        - If the definition is too large, the user must obtain large portions of the index,
        which may be unpalatable.

## Some Softness

For both Sourcify and 4Byte the hard criteria that define a volume are arbitrary and are
really just soft definitions that serve as schelling points.

If someone is aggregating contract metadata for the last complete volume (e.g., the
next volume in the database), they can decide if a contract will go in. They can censor,
forget, miss, or ignore pieces of data. That is okay because anyone can publish a
different version of that volume, or include the missed data in the next volume.

What matters is that someone makes the decision: This is going to be a collection of sealed
data.

## Some Bugs

Once sealed data is published, people can build upon that. In the event of a bad publisher,
good publishers need only step in and publish what they think is correct. The manifest
contract allows filtering by data type as well as pulbisher. The manifests themselves
can also contain versioning.

For example: TrueBlocks was publishing Unchained Index data and some incorrect data got in
to some chunks through a bug somewhere in Erigon. Firstly, the index immutability allowed
the bug to be identified. Second, the index was remade and republished. Post-bug chunk
hashes were now different, but the old ones were the same. Versioning in the manifest allows
communication of which software was used to build the index. So partial and complete
"rewritings of history" are practical, feasible and easy to navigate.

## Some Flattening

So the concept of volumes and chapters invokes a notion of nested directories, which makes
it easy for a user to know what to grab:

- Volume 1
    - Chapter 1 (Alice wants this)
    - Chapter 2
- Volume 2
    - Chapter 1 (Alice wants this)
    - Chapter 2

Yet the publishing and fetching happens in a flat manner. You can just get the chapters
by themselves. You start with a table with hashes to know how to get the chapters.

- Volume 1, chapter 1 -> You need hash A (Alice wants this)
- Volume 1, chapter 2 -> You need hash B
- ...
- Volume 2, chapter 1 -> You need hash X (Alice wants this)
- Volume 2, chapter 2 -> You need hash Y
- ...

So a local instantiation of the index/database can exist in any sort of format that
makes sense.

- Chapters nested in volumes
- Volumes nested in chapters
- Flat file structure
- Key-value database

The means that implementations that use the database/index are not restricted. A 4byte directory
could exist in one client as a database, and in another as a flat file structure.

## Some Conversion

The address-apearance-index is constructed with a folder structure like that
first groups by address, then by block range. The thinking is that someone might
want just one of these top level directories:

- chapter_0x00 (Alice wants this)
    - chapter_0x00_volume_000_000_000.ssz
    - chapter_0x00_volume_000_100_000.ssz
    - ...
    - chapter_0x00_volume_015_300_000.ssz
    - chapter_0x00_volume_015_400_000.ssz <-- Chapter from the latest volume.

So for the other databases/indices, Alice might have immutable files of the form:

- 4byte
    - volume_2017_Q1_chapter_0xff.csv
    - ...
    - volume_2022_Q1_chapter_0xff.csv
    - volume_2022_Q2_chapter_0xff.csv
- Sourcify
    - volume_2017_Q1_chapter_Qm47.json
    - ...
    - volume_2022_Q1_chapter_Qm47.json
    - volume_2022_Q2_chapter_Qm47.json

Though when creating the first volume it would be a very large file, you you could
try to sort data into rough time periods when they were incorporated.

Or if the volumes were defined by data size (like the Unchained Index), or buckets by count:

- 4byte
    - volume_000_000_chapter_0xff.csv
    - ...
    - volume_001_075_chapter_0xff.csv
    - volume_001_076_chapter_0xff.csv
- Sourcify
    - volume_000_000_chapter_Qm47.json
    - ...
    - volume_031_309_chapter_Qm47.json
    - volume_031_310_chapter_Qm47.json


I lay out a spec in case a database wants to drift in this direction:
[Time Ordered Distributable Database](#erc-time-ordered-distributable-database)

It is a generalisation of several databases.

## Some Manifesting

Each index/database needs a way to now what is in the index. As new pieces are added, the
old hashes are unchanged. The brilliant solution for this is to publish a hash in a smart
contract.

Not just any smart contract. One with no governance, no upgrades, no permissions, no votes.
Just a place to publish.

UnchainedIndex_v2.sol is deployed on mainnet and is ready to go.

Any index/database can use it!

Perhaps there is benefit to be had from making guidelines for how to publish a new manifest.
The publishing system could be oblivious to the type of data being published.

Maybe someone wanted to publish data in a similar way, but wanted a different contract
deployment. Yet they wanted people to read from it in the same way.

I lay out a spec in case someone wants to publish using UnchainedIndex's contract:
[Generic Attributable Manifest Broadcaster](#erc-generic-attributable-manifest-broadcaster)

Continue on to [poking part 7](part_7.md)

----
## References

### Unchained Index

The Unchained Index is a naturally-sharded, easily-shared, reproducible, and minimally-sized.
immutable index for EVM-based blockchains.

https://trueblocks.io/papers/2022/file-format-spec-v0.40.0-beta.pdf

### Sourcify

Sourcify enables transparent and human-readable smart contract interactions through automated Solidity contract verification, contract metadata, and NatSpec comments.

https://sourcify.dev/


### 4Byte directory

List of 4byte identifiers for EVM smart contract functions

https://www.4byte.directory/

https://github.com/ethereum-lists/4bytes

### Otterscan

A blazingly fast, local, Ethereum block explorer built on top of Erigon

https://github.com/wmitsuda/otterscan

### ERC-time-ordered-distributable-database

A format for useful peer-to-peer databases.

https://github.com/perama-v/TODD

### ERC-generic-attributable-manifest-broadcaster

A contract for announcing newly published metadata.

https://github.com/perama-v/GAMB

### address-appearance-index-specs

A specification of an index that maps addresses to the transactions that they appear in.

https://github.com/perama-v/address-appearance-index-specs
