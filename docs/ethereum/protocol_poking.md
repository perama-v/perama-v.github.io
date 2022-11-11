---
layout: page
title:  "Protocol poking"
permalink: /ethereum/protocol-poking
toc: true
---
```
Abstractions in pen;
Protocols for protocols.
Recipe for the glue
```
This post is mostly a wandering journey looking at impacts of "EIP-4444: History Expiry"
(a facet of The Purge), which necessitates alternative history access.
The Portal Network is positioned to provide that access, and I am interested in what are
the flow on effects of "sweeping away" history.

&#x2728;&#x1F9F9;&#x1F32A;

What will it be like to use a light-weight client?

What are some pieces that are important in the story?

I start out poking around with a few ideas. That leads to some code,
then to a spec, and a few other ideas. I hope you enjoy this informal journey with me.

Really this started with this kernel of an idea:
```sh
Introspection on ones own wallet is not really possible without a bit
of trust-hand-waving or a trusty ubuntu machine.

But it could be! The ingredients seem to be there at least.
```

If you like, these are the spoilers that came out of this:

- ["A Primitive User Interface"](#a-primitiveui)
- https://github.com/perama-v/address-appearance-index-specs
- https://github.com/perama-v/min-know
- https://github.com/perama-v/GAMB
- https://github.com/perama-v/TODD


Or proceed on with the exploration, ordered chronologically.


> prōtos "first" + kolla "glue"

---

- [Part I - Musing: Is there an underserved user in the history-expiry roadmap? (Yes, a regular wallet user)](#part-i---musing-is-there-an-underserved-user-in-the-history-expiry-roadmap-yes-a-regular-wallet-user)
  - [The friend](#the-friend)
  - [The search](#the-search)
  - [The light](#the-light)
  - [The dark](#the-dark)
  - [The grey](#the-grey)
  - [The idea](#the-idea)
- [Part II - Code: Can you divide a useful index into tiny useful parts? (Yes, prototype)](#part-ii---code-can-you-divide-a-useful-index-into-tiny-useful-parts-yes-prototype)
  - [A notepad](#a-notepad)
  - [A feature](#a-feature)
  - [A pivot](#a-pivot)
  - [A binary](#a-binary)
  - [A parser](#a-parser)
  - [A format](#a-format)
  - [A folder](#a-folder)
- [Part III - Specifying: Could other clients use a divided index? (Yes, with a spec)](#part-iii---specifying-could-other-clients-use-a-divided-index-yes-with-a-spec)
  - [Some Implementers](#some-implementers)
  - [Some Scope](#some-scope)
  - [Some Mimicry](#some-mimicry)
  - [Some Clarity](#some-clarity)
  - [Some Samples](#some-samples)
- [Part IV - Examining: Does a divided index result in anything useful? (Yes, personal history)](#part-iv---examining-does-a-divided-index-result-in-anything-useful-yes-personal-history)
  - [The Monotransaction](#the-monotransaction)
  - [The Toolbox](#the-toolbox)
  - [The LogJogger](#the-logjogger)
  - [The CodeRunner](#the-coderunner)
  - [The Pause](#the-pause)
  - [The Interrogation](#the-interrogation)
- [Part V - Remotes: Can remote resources make tx history human-readable? (Yes, through decoding and ABIs)](#part-v---remotes-can-remote-resources-make-tx-history-human-readable-yes-through-decoding-and-abis)
  - [A LocalFirst](#a-localfirst)
  - [A FourByte](#a-fourbyte)
  - [A Choice](#a-choice)
  - [A Review](#a-review)
  - [A Glimpse](#a-glimpse)
  - [A Trial](#a-trial)
  - [A Contractor](#a-contractor)
  - [A Sourcerer](#a-sourcerer)
  - [A User](#a-user)
  - [A Bool](#a-bool)
  - [A MatchMismatch](#a-matchmismatch)
  - [A Datum](#a-datum)
  - [A PrimitiveUI](#a-primitiveui)
  - [A Freshening](#a-freshening)
- [Part VI - Acquisitions: Can useful remote resources become distributed (Yes, by generalising a format)](#part-vi---acquisitions-can-useful-remote-resources-become-distributed-yes-by-generalising-a-format)
  - [Some Hunting](#some-hunting)
  - [Some MetadataTrials](#some-metadatatrials)
  - [Some Ideals](#some-ideals)
  - [Some Criteria](#some-criteria)
  - [Some TimeOrdering](#some-timeordering)
  - [Some Bookish](#some-bookish)
  - [Some Softness](#some-softness)
  - [Some Bugs](#some-bugs)
  - [Some Flattening](#some-flattening)
  - [Some Conversion](#some-conversion)
  - [Some Manifesting](#some-manifesting)
- [Part VII - Coalesce: Is a local, lighthweight personal history browser a good idea? (Plausibly so)](#part-vii---coalesce-is-a-local-lighthweight-personal-history-browser-a-good-idea-plausibly-so)
  - [Useful?](#useful)
  - [Robust?](#robust)
  - [Important?](#important)
  - [Novel?](#novel)
  - [Hard?](#hard)
  - [Untouched?](#untouched)
  - [Next?](#next)
  - [References](#references)
    - [Ethereum](#ethereum)
    - [Ethereum consensus specification](#ethereum-consensus-specification)
    - [EIP-4444](#eip-4444)
    - [Portal Network](#portal-network)
    - [trueblocks-core](#trueblocks-core)
    - [Unchained Index](#unchained-index)
    - [SSZ Spec](#ssz-spec)
    - [Snappy](#snappy)
    - [IPFS CID](#ipfs-cid)
    - [ERC-time-ordered-distributable-database](#erc-time-ordered-distributable-database)
    - [ERC-generic-attributable-manifest-broadcaster](#erc-generic-attributable-manifest-broadcaster)
    - [address-appearance-index-specs](#address-appearance-index-specs)
    - [Min-know](#min-know)
  - [Thanks :)](#thanks)


# Part I - Musing: Is there an underserved user in the history-expiry roadmap? (Yes, a regular wallet user)

- Hypothesis: There is an underserved user in the roadmap.
- Methods: Examine actions and requirements of a single wallet user.
- Conclusion: A single wallet user lacks knowledge required to query networks in the history-expiry roadmap.

## The friend

Lets say you made a friend get a wallet and you sent them some thing.

Now a year later, they suddenly are interested and want to find out what they have.

They have their address, a computer and they like the idea that whatever you sent them
is part of some trustless peer-to-peer future. Let's do it, they say!

"What did you send me?". You can't recall, but no matter, they have their address.
Just use a node to work it out.

## The search

The problem is, that the magical EVM can spit out all sorts of things, being general purpose.
So the details involving your friend's address could be halfway into some complex nested
function call. Especially if they went wandering into some unusual contracts, or
were the subject of the actions of a third parties' transaction.

A node can't really just give you that information, and it is a gnarly sort of problem
that pops up here and there. So with a full node, or even if you have a network where peers
have shared partial nodes (the excellent portal network), it doesn't advance you in the quest
of knowing which transactions are of interest.

You have to know what you want before talking to the node.

## The light

The only way to find out what an address has been up to on the chain is to run a
trace of every single transaction ever and look for it. So that is fun, but
some work. The neat thing is, that it can be performed once, the results for all
addresses scraped and stored, then shared with others. TrueBlock's software
[trueblocks-core](#trueblocks-core)
does this and produces the [Unchained Index](#unchained-index).
Now all the transactions that even mention your address can be found quickly.

## The dark

Yet the index, being so powerful, is about 80GB. Fine, that's tiny in node-land,
where a real meaty data-science node is about the size of a 2 terabyte hard drive + 1 byte.
It is not a complaint, [Erigon](#erigon) has done tremendous work bringing
archive nodes to many home users (and they have more upgrades in the pipeline).
Though users from laptop-land and mobile-land might not stomach that with joy.

So what is to be done? Does the friend fetch the whole thing?

No! They can just fetch the ~3GB of bloom filters, then they can fetch the
bits of the index that matters to them. So 3GB plus some 25MB chunks of the index.
If they have been busy on chain, maybe it starts to creep up. Overall, pretty good.
Plus those chunks can be shared and torrent-seeder-style for index-network-health.

It's a really good direction, and honestly a really nice tradeoff. It could bolster
the Unchained Index, which is more of a cornerstone than appears to be common knowledge.

## The grey

Is that the best we can do? Yes, no, yes, no - I think we can go smaller, without
undermining the Unchained Index.

I keep having an idea! It's not that the idea is great. More that it is recurring,
so I think I will write it all down and have a play. It's a sort of protocol, which
is nice, and might be useful without being a burden, which is also good.

## The idea

I'll try piecing together a system for splitting the Unchained Index into pieces that
are more appetising for a user-land user.

The idea is:
```
Given that a user starts with an address in mind, can they just fetch a piece
that contains everything for addresses that are similar to theirs?
```
Our friend, the peer, says to their peer:
```
My address starts with 0xc4, so give me index data that only has 0xc000... to 0xcfff...
```
zero to nine then a to f gives us 16. Combinatorialise that, and we have 16**2 = 256
"startings".

So really the idea is: take the 80GB and split it into ~350MB parts that can be
"taken or leaven" (left to ferment).

## The context

This is all in the setting of the [Ethereum](#ethereum) roadmap becoming
gradually more likely to implement history expiry ([EIP-4444](#eip-4444)).
This allows nodes to be teeny-weeny, which is nice for users.

In this world, a separate network such as the [Portal Network](#portal-network) is used to
"access the chain". This means send transactions and look things up such as "what is the
balance for my address in this contract?".

However, as we have just discovered, it is not very easy to answer the question:

"What has my wallet been up to?"

Which may seem like an odd request. Yet, I feel, with some thought, it is a cornerstone
for a delightful user interface. Without delight, we have no users.

What is the setup for a "power browser"?:

- Present day:
    - Node: [Erigon](#erigon)
    - Index: [Unchained Index](#unchained-index)
    - Interface: [Otterscan](#otterscan)
- Post EIP-4444: ?
    - Node: [A portal node](#portal-network)
    - Index: [address-appearance-index](#address-appearance-index-specs)
    - Interface: A primitive UI fuelled by distributed [4byte](#4byte-directory) and
    [Sourcify](#sourcify) databases.


# Part II - Code: Can you divide a useful index into tiny useful parts? (Yes, prototype)

- Hypothesis: The Unchained Index can be divided into tiny functional parts.
- Methods: Createa a rust library that transforms the index into a new format.
- Conclusion: Dividing addresses by first two hex characters results in <500MB parts.

## A notepad

The ideas are here and there. They form into a rambling idea.md then a few bulky
this_part.md and that_part.md.

The idea is to not only have the idea and flesh it out, but have it documented along
the way. It quickly becomes a fair bit of text and there are some design decisions
to be played with. I'll spare you from reading it.

Never mind polishing it up I thought, I get on to the code to try it out.

## A feature

"I'll build it into trueblocks-core", I think. That will avoid duplicating work
handling the index `chunk.bin` files, and will be a nice way to have it be available to more users.

Something like:
```
chifra chunks export reshaped-index
```

So I am in the go code and living dangerously in the between where the CLI meets the
function entry points. It's a bit slow going, a few Golang tabs open. Making a little progress
building a mental model for how the codegen pieces fit together.

Then I realise, this could be a burden on true-blocks core. A semi-parasitic feature that is
sort of tacked on for another purpose. Would it be a pain to maintain, and a stain on
their `main()`?

## A pivot

What if the index is handled directly? Trueblocks-core races through a local tracing erigon
node with `chifra scrape`. That makes the index, and keeps it up to date. It also
can push the index to IPFS, where it can be fetched as a standalone.

I can do the conversion separately! This separates the concept so that if it is a bad
idea, it hasn't wound up bloating trueblocks-core.

Something like: `unchained_index_eater.rs`

## A binary

Now I'm in `~/.local/share/trueblocks/mainnet/finalized` with thousands
of `10293949-10294201.bin`-looking files. The Unchained Index white paper is out
and we are going to get the data directly.

The magic number, the size layout descriptions, the expected bytes, the addresses,
block numbers, indices and their offsets.

## A parser

So I reader that can skip around to the different parts of the index `chunk.bin`
files. The unchained specification outlines precisely where the different bytes
all start and stop. I copy the spec and start piping values into buffers.

Its magic

```rust
let mut magic: [u8; VAL] = [0; VAL];
rdr.read_exact(&mut magic)?;
if magic != MAGIC {
    return Err(anyhow!("file {:?} has incorrect magic bytes", path));
}
```
The beautiful thing about rust is that it doesn't make a fuss when you don't know something.
It merely teaches you, right there and then. Things you thought you understood are exposed
and quickly patched up. Its very nice.

The chunk files are organised in a table with addresses up top, and transaction identifiers down
the bottom. So the idea is to be able to spawn a "hunter" with a family of addresses in mind.
It runs down the top table, and when a relevant address pops up, it darts to the transactions
table and gobbles up the transactions. Those can then be wrangled into a new database.

    /// 1. Iterate over address entries, starting reader at the address table.
    /// 2. For current address entry, read the address, offset and count.
    /// 3. Determine jump location using offset and count.
    /// 4. Jump to the appearance table.
    /// 5. Read and store transactions in vector, looping until count satisfied.
    /// 6. Skip transactions outside desired RANGE.
    /// 7. Save to transactions to database, adding to existing AddressData for that address.
    /// 8. Update address byte index for the next entry
    /// 9. Jump back to address table, go to 2.

There are improvements to be made to save time and memory, but not now:
There are Hypotheses to be Methodised.

## A format

So how to store this new data? A furrow-inducing topic. We have disk footprints and network
transmission, as well as the potential for multiple languages to interact with the data.

Learning from the Unchained Index, a good amount of speed and compactness can be had from
laying out values in an offset table. Follow the document, read files by seeking and jumping
to the different bytes that are relevant. Hashes in a manifest provide a sanity check
on the integrity of the index data.

Yet I also recall that a great deal of effort went into the consensus specification
serialization work. [SSZ](#ssz-spec) allows complex containers of strictly typed data to be
serialized and hashed. Could the smallest pieces of data in the new index be `.ssz-ified`
to ensure that the goal "small distributed parts" is achieved?

The properties seem perfect:

- Injective: A serialized object has only one possible deserialization output.
- Surjective: A serialized object always has something it can be deserialized to.
- Hashable: Tree root hash / merkle hashing is available for rich unique identification of parts.
- Offsets: For reading larger containers efficiently.
- Compressible: Snappy compression standardised for network transmission.

So I run a little test and find that the data encoded as [ssz_snappy](#snappy) is quite small.
With some extrapolation, it seems approximately the same size as the Unchained Index (<80GB).

That's good. In fact, it makes me think that it could end up smaller.

If addresses occur frequently, every chunk file will save the address anew. Perhaps
some addresses are being duplicated, perhaps
hundreds or thousands of times. At 20 bytes per address, this could add up. We will see though,
as it may be that the general purpose nature of ssz has some inefficiencies versus
a custom offset table.

## A struct-uring

Now [lighthouse et al](#lighthouse), have done some amazing heavy lifting with respect to
ssz. Just look at these neat little `Encode`, `Decode` and `TreeHash` add-ons that
equip this struct with the ability to be ssz-encoded.

```rust
#[derive(PartialEq, Debug, Encode, Decode, Clone, TreeHash)]
pub struct AddressIndexVolumeChapter {
    /// Prefix common to all addresses that this data covers.
    pub address_prefix: FixedVector<u8, DEFAULT_BYTES_PER_ADDRESS>,
    /// The blocks that this chunk data covers.
    pub identifier: VolumeIdentifier,
    /// The addresses that appeared in this range and the relevant transactions.
    pub addresses: VariableList<AddressAppearances, MAX_ADDRESSES_PER_VOLUME>,
}
```

## A guillotine

The database is to be broken up at some cadence. The Unchained Index targets 2,000,000
"appearances" (an instance of an address appearing during the trace of a transaction), which
amounts to ~25MB files being produced every ~12 hours.

I spend a great many rambling markdown notes outlining reasons and rationales for how to organise
the new data. With a finger to the air, it seems good to publish every 2 weeks, and have
individual users be acquiring a 256th of the total database.

So every 100_000 blocks a guillotine falls and any `chunks.bin` file that falls within this
range is harvested and the transactions it indexes are sorted into 256 different files. One
for every address starter: `0x00`, `0x01`, ... , `0xff`.

## A foldering

I set up the prototype and fuss around making things work for different setups. What
would the path be for the directory for addresses starting with `0xf6` on each
of the different operating systems? If someone else were to test out this prototype,
how would they source the data? What if they wanted to use the full Unchained Index data
but didn't use the default path that trueblocks-core sets up? I think I got a pretty
good system going to automate these things.

So the example data consists of four Unchained Index chunk files (4 * 25MB = 100MB)
that are harvested and converted into address-appearance-index files.

From:
```
- ./min-know/data/samples/
    - /unchained_index_mainnet/ (100MB "before")
        - 011283653-011286904.bin (25MB)
        - 012387154-012389462.bin
        - 013408292-013411054.bin
        - 014482581-014485294.bin
    - /address-appearance-index-mainnet/ (86 MB "after")
        - chapter_0x00/
            - chapter_0x00_volume_011_200_000.ssz_snappy (332KB)
            - chapter_0x00_volume_012_300_000.ssz_snappy
            - chapter_0x00_volume_013_400_000.ssz_snappy
            - chapter_0x00_volume_014_400_000.ssz_snappy
        - ...
        - chapter_0xff/
            - chapter_0xff_volume_011_200_000.ssz_snappy
            - chapter_0xff_volume_012_300_000.ssz_snappy
            - chapter_0xff_volume_013_400_000.ssz_snappy
            - chapter_0xff_volume_014_400_000.ssz_snappy
```

Take that last file: 014_400_000. It accepts any transaction in blocks

- 14_400_000 to 14_499_999.

This means that it contains all the data from the Unchained Index chunk `.bin` file spanning:

- 14_482_581 to 14_485_294.

So as a feasibility analysis, the new index files contain all the data that the chunk
files contain, but are a tad smaller. This is despite having nested struct information
in each of the .ssz_snappy. It seems like the snappy compression does a good job at removing
this overhead. I estimate that the .ssz_snappy files will be ~2MB on average

There are also likely to be savings for addresses that appear frequently. Duplication
must occur at the 12 hour mark for the Unchained Index, and at the 2 week mark for the
address-appearance-index. It remains to be tested though.

I call the prototype min-know. &#x1F4D8;&#x1F50D;&#x1F41F;

# Part III - Specifying: Could other clients use a divided index? (Yes, with a spec)

- Hypothesis: The address-appearance-index can be described without prototype-specific
or network-specific details.
- Methods: Create a specification for the index using ssz, making decisions and parameters explicit.
- Conclusion: Specification can be described, allowing for alternate implementations and networks.

## Some Origins

Consider that if the index is useful, a recipe for integrating it would be good.
How should the recipe be formed?

With what `kolla` should the `prōtos` page be adhered with?

## Some Scope

Now I am thinking about what the assumptions about the world that the index spec makes.
Additionally, wwhat are the assumptions that someone can make about the index.

What does a good specification have? I try not to spend too much time on this.
For the idea of this index itself may be poorly formulated, and a robust specification
will be a waste.

## Some Mimicry

I see that the python spec has been working well for the [beacon chain](#beacon-chain) and the
[portal network](#portal-network), and like the format.

Perhaps the target audience for this new address-appearance-index-spec will be familiar
with this format?

## Some Clarity

The process of writing out the spec helps to decide what are parameters of the system,
and how they are decided. E.g., What is the maximum possible length of a list of items
that just keeps growing? How many bytes should your network name occupy? Strings
are abolished. There are no vagaries here.

I write down the spec in python and then represent each on as a struct in rust, seasoning
with some ssz-derivers. I don't go as far as making the spec executable. The process
writing a script to extract the python looks to be a chore, and so I leave it open
for a few bugs.

Transactions:
```python
class AppearanceTx(Container):
    block: uint32
    index: uint32
```
List of transactions:
```python
class AddressAppearances(Container):
    address: ByteVector[DEFAULT_BYTES_PER_ADDRESS]
    appearances: List[AppearanceTx, MAX_TXS_PER_VOLUME]
```
List of lists of transactions:
```python
class AddressIndexVolumeChapter(Container):
    address_prefix: ByteVector[NUM_COMMON_BYTES]
    identifier: VolumeIdentifier
    addresses: List[AddressAppearances, MAX_ADDRESSES_PER_VOLUME]
```
List of lists of lists of transactions:
```python
class AddressChapter(Container):
    identifier: ChapterIdentifier
    members: List[AddressIndexVolumeChapter, MAX_VOLUMES]
```
**Vector** of lists of lists of lists of transactions:
```python
class AddressAppearanceIndex(Container):
    chapters: Vector[AddressChapter, NUM_CHAPTERS]
```

There it is. The final container. It seems silly that you can lay it out like this,
and have the final data be so compact.

I group the data by Chapters (address groups) rather than Volumes (time-based groups)
because that is what the user wants. So it is more natural to have a directory or database
for each Chapter than each Volume.

# Part IV - Examining: Does a divided index result in anything useful? (Yes, personal history)

- Hypothesis: The information is useful for a regular user.
- Methods: Use local Ethereum node as if it were a Portal Node, use the index to get transaction receipts relevant to the user.
- Conclusion: The receipts for an address are obtainable but not immediately useful without
additional work.

## The Practicality

So now there is a way to punch in an address and out comes a list of transaction
identifiers.

```
0xabcd...1234

> block_number_a, transaction_index_x
> block_number_b, transaction_index_y
> block_number_c, transaction_index_z
```

 What can be done with that? Is this a pretend solution that doesn't really
lead to anything tangible for a normal person?

We will try to use the information we have obtained from this new index. Think
of this as a ~350MB thingy that was downloaded over IPFS for the user. Their wallet
knows what their main address starts with, and initiated the download for the appropriate
data.

Also imagine that they have a Portal Node. It holds some data, defined by the node's ID and
some resource settings like "don't take up more than x space on my laptop".

We will for now connect to a regular local node over `http://localhost:8545`, and pretend
it is a [Trin Portal Client](#trin) by limiting our requests to those that the Portal Network
will support.

## The Toolbox

There are many supported methods listed in the [Portal Network spec](#portal-network),
but these catch my eye for our intended purpose:

- `eth_getTransactionByBlockAndIndex(block_number, transaction_index)`
    - Returns tx_hash (then go to `eth_getTransactionReceipt`)
- `eth_getTransactionReceipt(tx_hash)`
    - Has logs, which has topics, data fields and contract contexts (then go to `eth_getCode`).
    - Has revert status and gas used.
- `eth_getCode`
    - Returns the runtime bytecode which contains source code metadata, which
        can be used to get a link to the contract abi.

## The Monotransaction

Given a single transaction `eth_getTransactionByBlockAndIndex` delivers, we
can have a look and see if it is useful for a normal user.

I pick a random address that appears in a block that the sample data covers.

```
0x846be97d3bf1e3865f3caf55d749864d39e54cb9
```
In [min-know](#min-know) I set up an example that extracts and prints the data.
```
cargo run --example user_0_find_transactions
```
It finds two transactions for this user:
```
Txs [
    AppearanceTx {
        block: 12387154,
        index: 312,
    },
    AppearanceTx {
        block: 12387161,
        index: 166,
    },
]
```
Another example picks the first transaction and ask the node about it:
```
cargo run --example user_1_transaction_receipt
```
It finds that there are 5 logs. Here they are below, feel free to skip past them. It is
clear that a normal user is not going to be interested.
```
Transaction gas used: Some(123808)
Transaction logs: [
    Log {
        address: 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2,
        topics: [
            0xe1fffcc4923d04b559f4d29a8bfc6cda04eb5b0d3c460751c2402c5c5cc9109c,
            0x0000000000000000000000007a250d5630b4cf539739df2c5dacb4c659f2488d,
        ],
        data: Bytes(
            [
                0,
                0,
                0,
                ...
                142,
                0,
                0,
            ],
        ),
        block_hash: Some(
            0x6ef56e42a79d6732c770cbbbf2f00e2d05729c8b450351fff8912dc8191c4951,
        ),
        block_number: Some(
            12387154,
        ),
        transaction_hash: Some(
            0x1a8d94dda1694bad33384215bb3dc0a56652b7069c71d2b1afed35b24c9b54df,
        ),
        transaction_index: Some(
            312,
        ),
        log_index: Some(
            346,
        ),
        transaction_log_index: None,
        log_type: None,
        removed: Some(
            false,
        ),
    },
    Log {
        address: 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2,
        topics: [
            0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef,
            0x0000000000000000000000007a250d5630b4cf539739df2c5dacb4c659f2488d,
            0x0000000000000000000000001636a5dfcf7a21945c06d1bea40b52ce975ea614,
        ],
        data: Bytes(
            [
                0,
                0,
                0,
                ...
                142,
                0,
                0,
            ],
        ),
        block_hash: Some(
            0x6ef56e42a79d6732c770cbbbf2f00e2d05729c8b450351fff8912dc8191c4951,
        ),
        block_number: Some(
            12387154,
        ),
        transaction_hash: Some(
            0x1a8d94dda1694bad33384215bb3dc0a56652b7069c71d2b1afed35b24c9b54df,
        ),
        transaction_index: Some(
            312,
        ),
        log_index: Some(
            347,
        ),
        transaction_log_index: None,
        log_type: None,
        removed: Some(
            false,
        ),
    },
    Log {
        address: 0x106d3c66d22d2dd0446df23d7f5960752994d600,
        topics: [
            0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef,
            0x0000000000000000000000001636a5dfcf7a21945c06d1bea40b52ce975ea614,
            0x000000000000000000000000846be97d3bf1e3865f3caf55d749864d39e54cb9,
        ],
        data: Bytes(
            [
                0,
                0,
                0,
                ...
                132,
                118,
                220,
            ],
        ),
        block_hash: Some(
            0x6ef56e42a79d6732c770cbbbf2f00e2d05729c8b450351fff8912dc8191c4951,
        ),
        block_number: Some(
            12387154,
        ),
        transaction_hash: Some(
            0x1a8d94dda1694bad33384215bb3dc0a56652b7069c71d2b1afed35b24c9b54df,
        ),
        transaction_index: Some(
            312,
        ),
        log_index: Some(
            348,
        ),
        transaction_log_index: None,
        log_type: None,
        removed: Some(
            false,
        ),
    },
    Log {
        address: 0x1636a5dfcf7a21945c06d1bea40b52ce975ea614,
        topics: [
            0x1c411e9a96e071241c2f21f7726b17ae89e3cab4c78be50e062b03a9fffbbad1,
        ],
        data: Bytes(
            [
                0,
                0,
                0,
                ...
                251,
                144,
                101,
            ],
        ),
        block_hash: Some(
            0x6ef56e42a79d6732c770cbbbf2f00e2d05729c8b450351fff8912dc8191c4951,
        ),
        block_number: Some(
            12387154,
        ),
        transaction_hash: Some(
            0x1a8d94dda1694bad33384215bb3dc0a56652b7069c71d2b1afed35b24c9b54df,
        ),
        transaction_index: Some(
            312,
        ),
        log_index: Some(
            349,
        ),
        transaction_log_index: None,
        log_type: None,
        removed: Some(
            false,
        ),
    },
    Log {
        address: 0x1636a5dfcf7a21945c06d1bea40b52ce975ea614,
        topics: [
            0xd78ad95fa46c994b6551d0da85fc275fe613ce37657fb8d5e3d130840159d822,
            0x0000000000000000000000007a250d5630b4cf539739df2c5dacb4c659f2488d,
            0x000000000000000000000000846be97d3bf1e3865f3caf55d749864d39e54cb9,
        ],
        data: Bytes(
            [
                0,
                0,
                0,
                ...
                0,
                0,
                0,
            ],
        ),
        block_hash: Some(
            0x6ef56e42a79d6732c770cbbbf2f00e2d05729c8b450351fff8912dc8191c4951,
        ),
        block_number: Some(
            12387154,
        ),
        transaction_hash: Some(
            0x1a8d94dda1694bad33384215bb3dc0a56652b7069c71d2b1afed35b24c9b54df,
        ),
        transaction_index: Some(
            312,
        ),
        log_index: Some(
            350,
        ),
        transaction_log_index: None,
        log_type: None,
        removed: Some(
            false,
        ),
    },
]
```

## The Implications

That receipt is pretty opaque. Can we do better? I think so.

What have have retrieved are `Events`
that have been hand crafted by the contract engineers. They are designed to fire off
when very important things happen. It is convention to use events to keep track of
things, and they are often specified.

Just take a look at the [ERC-20 token standard](https://eips.ethereum.org/EIPS/eip-20).
Under the events headig, It states that:

```
Transfer

MUST trigger when tokens are transferred, including zero value transfers.

A token contract which creates new tokens SHOULD trigger a Transfer event with the _from address set to 0x0 when tokens are created.

event Transfer(address indexed _from, address indexed _to, uint256 _value)
```

So events are going to be useful, and are probably at the right level that a user would
be interested in. Something like: "You have been transferred x of y tokens from z address"
is the goal here.

If standardised contracts MUST emit certain events, and we can access those events, then
that is a reliable way to examine one aspect of chain history.

## The LogJogger

The first useful thing that a user could want in a transaction is knowing the name
of the events.

Keeping in mind their address may be the initiator, recipient, or some strange
by-product during EVM execution.

So let us look at the first `log`, and examine the first element in `topics`. The value
in `topic[0]` is always reserved for the "event signature". It is the hash of the
string of the event.

```
Transaction logs: [
    Log {
        address: 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2,
        topics: [
            0xe1fffcc4923d04b559f4d29a8bfc6cda04eb5b0d3c460751c2402c5c5cc9109c,
            0x0000000000000000000000007a250d5630b4cf539739df2c5dacb4c659f2488d,
        ],
```
This is wonderful. Usually people only refer to these by the first four bytes:

`0xe1fffcc4`

The log also contains a contract address `0xc02a...6cc2`. This is the contract that
emitted the event in the log! This is the next piece of the puzzle.

## The CodeRunner

We now call our node again with the contract address from any log we get

`eth_getCode(contract_address)`

That returns a blob of bytecode. This code is the "program for the transaction", but the
[solidity compiler](#https://docs.soliditylang.org/en/latest/metadata.html#contract-metadata)
automatically includes additional information. This information is metadata that
is tacked onto the end. Best seen in this playground
(https://playground.sourcify.dev/) created by [Sourcify](#sourcify).

What if we could snip off that metadata and go hunting for the contract? It might contain
more useful information for us.

## The SeaBorer

So by reading the length of the metadata, then snipping it off, we have the Concise
Binary Object Representation (CBOR) which potentially has an identifier we
can use to look for the contract ABI. For example an
Interplanetary File System (IPFS) [Content Identifier (CID)](#ipfs-cid).

Here's a little test function that does that.

```rust
fn cid_extraction() {
    let test_metadata = "a2646970667358221220c019e4614043d8adc295c3046ba5142c603ab309adeef171f330c51c38f1498964736f6c6343000804";
    let bytes = hex::decode(test_metadata).unwrap();
    let cid = ipfs_cid_from_metadata(&bytes).unwrap();
    assert_eq!(
        cid,
        Some(String::from(
            "QmbGXtNqvZYEcbjK6xELyBQGEmzqXPDqyJNoQYjJPrST9S"
        ))
    );
}
```

CIDv0 all start with "Qm", so that's the good stuff.

## The Pause

So far we have:

1. `portal_network(0x84)` -> chapter_0x84 (~350MB of the index for address starting with `0x84`)
2. `chapter_0x84` -> block_num, tx_index
3. `eth_getTransactionByBlockAndIndex(block_num, tx_index)` -> tx_hash
4. `eth_getTransactionReceipt(tx_hash)` -> log_topic and contract_address
5. `eth_getCode(contract_address)` -> bytecode, bytecode_metadata


## The Interrogation

So a single transaction now can be interrogated against our portal node.

```sh
cargo run --example user_2_inspect_transaction_logs
```
Reveals:
```
Tx 0x1a8d94dda1694bad33384215bb3dc0a56652b7069c71d2b1afed35b24c9b54df has 5 logs

Contract: 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
        Topic logged: 0xe1fffcc4923d04b559f4d29a8bfc6cda04eb5b0d3c460751c2402c5c5cc9109c

Contract: 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
        Topic logged: 0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef

Contract: 0x106d3c66d22d2dd0446df23d7f5960752994d600
        Topic logged: 0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
        IPFS metadata CID: "QmZwxURkw5nD5ZCnrhqLdDFG1G52JYKXoXhvvQV2e6cmMH"

Contract: 0x1636a5dfcf7a21945c06d1bea40b52ce975ea614
        Topic logged: 0x1c411e9a96e071241c2f21f7726b17ae89e3cab4c78be50e062b03a9fffbbad1

Contract: 0x1636a5dfcf7a21945c06d1bea40b52ce975ea614
        Topic logged: 0xd78ad95fa46c994b6551d0da85fc275fe613ce37657fb8d5e3d130840159d822
```

Not pretty to the human eye, but it is a start. We have the transactions for a user
and some contracts and topics they emit. Plus, look at that nicely decoded
IPFS hash! The other contracts did not have one - but I did not try for a swarm hash.
There may be other creative ways to get ABIs too...

Missing: Human readable translation of these transactions.


# Part V - Remotes: Can remote resources make tx history human-readable? (Yes, through decoding and ABIs)

- Hypothesis: Additional remote databases can make retrieved history useful for a normal user.
- Methods: For retrieved transaction receipts, call external 4byte and sourcify APIs and decode data.
- Conclusion: 4byte and sourcify provide sufficient data for a user to understand their wallet activity.

## A LocalFirst

So far we have only used information from our portal node and the index.

Next we need more information. This can go two ways:

- APIs (unhealthy)
- Corralling the data locally (healthy)

We will start with an API as a test, and then find a better way.

## A FourByte

Now we need an event signature database. People have been dutifully computing and collecting
these. So later we could give the database to everyone (TBD DB size). For
now we skip that and look it up in [4byte directory]`www.4byte.directory` /
https://github.com/ethereum-lists/4bytes

In goes `0xe1fffcc4` out comes:


|ID | Text Signature | Bytes Signature |
|-|-|-|
| 170346 | Deposit(address,uint256)	| 0xe1fffcc4 |

So the 170346th entry (there are nearly 1 million) is the one we are after.

This applies to the events but also any functions. Events have the full hash,
as we saw. But when decompiling raw bytecode, only the first 4 bytes are stored. So
that is where the directory name comes from.

## A Choice

We seem to have enough
here to avoid the hash collision that normally plagues function signatures.

Consider the topic:
```
0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
```
We use the four bytes to look up in the 4byte directory:
```
0xddf252ad
```
This could be one of the three options in the 4 byte registry:
```
BeckerrJon(bytes16,bytes4,bytes23,bytes16,bytes1,bytes7,bytes1)

join_tg_invmru_haha_c0bbdb6(uint256,uint256,uint256)

Transfer(address,address,uint256)
```

Mirror, mirror, on the wall?

However, we have the rest of the bytes from our log!

So we can check each one:
```
keccak("BeckerrJon(bytes16,bytes4,bytes23,bytes16,bytes1,bytes7,bytes1)")
ddf252ade8bdc6f18de3868ae50ab6e67ee261b7136b3141cd791f1ad4786a79
nope")

keccak("join_tg_invmru_haha_c0bbdb6(uint256,uint256,uint256)
78ffd931d380887ef41bca9957337adc0cba476e6219aa82096e5b085092b8f3
nope")

keccak("Transfer(address,address,uint256)
ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
yep")
```
So while the database contains "duds", we can exclude them with full certainty for events
(not for functions).

## A Review

So far we have:

1. `portal_network(0x84)` -> chapter_0x84 (~350MB db for address starting with `0x84`)
2. `chapter_0x84` -> block_num, tx_index
3. `eth_getTransactionByBlockAndIndex(block_num, tx_index)` -> tx_hash
4. `eth_getTransactionReceipt(tx_hash)` -> log_topic, contract_address
5. `eth_getCode(contract_address)` -> bytecode, bytecode_metadata
6. `decoder(bytecode_metadata)` -> ipfs_cid_for_abi
7. `4byte(topic)` -> event_name
8. `sourcify(contract_address)` -> abi

## A Glimpse

If we stopped there, we could have something quite nice. A list of transactions
and a super vague function name for them:

- Tx 0: `Confirmation(address,bytes32)`
- Tx 1: `runTrade(bytes)`
- Tx 2: `joinPool(address,uint256,uint256[2])`
- Tx 3: `swapExactTokensForTokens(uint256 amountIn, uint256 amountOutMin, address[] path, address to, uint256 deadline)`

Which is a sort of vague reminder of what has been happening for your address.
Unless there are some tricky function names that are not very helpful.

What did you confirm? What pool did you join? We need more, I think. We will try,
but first let us see how far this gets us to "Useful for a regular user".

It is possible that we need the full trace of the transaction. That is a big topic
and maybe worthwhile. It would invovle chasing down contracts, getting blocks,
checkin state, repeating - all the way through the EVM execution including nested
calls to other contracts.

Howevery I thinkg that that may end up being less helpful for your average wallet-goer.

Less is more? Hmm.

## A Trial

We continue with out random address `0x846be97d3bf1e3865f3caf55d749864d39e54cb9`
that appears in a block present in our sample index files.

Following the steps above, peering into `chapter_0x84` to get transaction ids, then
calling our local node twice (`eth_getTransactionByBlockAndIndex` then
`eth_getTransactionReceipt`), we can get the following information:

The event signatures are for the moment from the 4byte registry.


```
Tx 0x1a8d94dda1694bad33384215bb3dc0a56652b7069c71d2b1afed35b24c9b54df has 5 logs

At contract: 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
        topic logged: Some(0xe1fffcc4923d04b559f4d29a8bfc6cda04eb5b0d3c460751c2402c5c5cc9109c)
        Event: Deposit(address,uint256)

At contract: 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
        topic logged: Some(0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef)
        Event: Transfer(address,address,uint256)

At contract: 0x106d3c66d22d2dd0446df23d7f5960752994d600
        topic logged: Some(0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef)
        Event: Transfer(address,address,uint256)

At contract: 0x1636a5dfcf7a21945c06d1bea40b52ce975ea614
        topic logged: Some(0x1c411e9a96e071241c2f21f7726b17ae89e3cab4c78be50e062b03a9fffbbad1)
        Event: Sync(uint112,uint112)

At contract: 0x1636a5dfcf7a21945c06d1bea40b52ce975ea614
        topic logged: Some(0xd78ad95fa46c994b6551d0da85fc275fe613ce37657fb8d5e3d130840159d822)
        Event: Swap(address,uint256,uint256,uint256,uint256,address)
```

Do you know what? I dare say this might be a swap transaction on some sort of decentralised
exchange!

## A Contractor

In the above logs, we find a few addresses. What can be done about them?

Well, the portal network supports `eth_getCode`, which retrieves the "program"
that each of these topics originate from. Maybe we can use the contract metadata
to find some useful information.

## A Sourcerer

We snip off the final bytes of the contract code and do a little de-CBORing.
Then we decode that as an IPFS hash. Fetch that, and we have a contract ABI!

Maybe we can fill in the blanks for any missing 4byte entries?

The contract is going to have docs, and the contract name is bound to be there.
If the contract is not there, then we may be in the dark still.

I try to use the IPFS hash to get the data, but its coming up blank.

For now, we use the Sourcify API to get the contract ABI. They actually want the contract
address, not the IPFS hash. More on this later maybe.

For now, here it is. I have cut out the functions, because we don't need them at the moment.
```json
{
  "compiler": {
    "version": "0.4.19+commit.c4cbbb05"
  },
  "language": "Solidity",
  "output": {
    "abi": [
      {
        "constant": true,
        "inputs": [],
        "name": "name",
        "outputs": [
          {
            "name": "",
            "type": "string"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
      },
      ...
      {
        "payable": true,
        "stateMutability": "payable",
        "type": "fallback"
      },
      {
        "anonymous": false,
        "inputs": [
          {
            "indexed": true,
            "name": "src",
            "type": "address"
          },
          {
            "indexed": true,
            "name": "guy",
            "type": "address"
          },
          {
            "indexed": false,
            "name": "wad",
            "type": "uint256"
          }
        ],
        "name": "Approval",
        "type": "event"
      },
      {
        "anonymous": false,
        "inputs": [
          {
            "indexed": true,
            "name": "src",
            "type": "address"
          },
          {
            "indexed": true,
            "name": "dst",
            "type": "address"
          },
          {
            "indexed": false,
            "name": "wad",
            "type": "uint256"
          }
        ],
        "name": "Transfer",
        "type": "event"
      },
      {
        "anonymous": false,
        "inputs": [
          {
            "indexed": true,
            "name": "dst",
            "type": "address"
          },
          {
            "indexed": false,
            "name": "wad",
            "type": "uint256"
          }
        ],
        "name": "Deposit",
        "type": "event"
      },
      {
        "anonymous": false,
        "inputs": [
          {
            "indexed": true,
            "name": "src",
            "type": "address"
          },
          {
            "indexed": false,
            "name": "wad",
            "type": "uint256"
          }
        ],
        "name": "Withdrawal",
        "type": "event"
      }
    ],
    "devdoc": {
      "methods": {}
    },
    "userdoc": {
      "methods": {}
    }
  },
  "settings": {
    "compilationTarget": {
      "WETH9.sol": "WETH9"
    },
    "libraries": {},
    "optimizer": {
      "enabled": false,
      "runs": 200
    },
    "remappings": []
  },
  "sources": {
    "WETH9.sol": {
      "keccak256": "0x4f98b4d0620142d8bea339d134eecd64cbd578b042cf6bc88cb3f23a13a4c893",
      "urls": [
        "bzzr://8f5718790b18ad332003e9f8386333ce182399563925546c3130699d4932de3e"
      ]
    }
  },
  "version": 1
}
```
Does that look useful?

There are some goodies in there I think! What can you see?

## A User

We now have the ingredients that a frontend could use to cook up a nice meal (user interface
for understanding these transactions).

Note that the variable input names can be obtained from the contract ABI. This makes
the events more readable, because an event might be emitting a "`Transfer(address,address)`",
that you can then translate with confidence to determine what the meaning of each variable is,
such as "`Transfer(to address, from address)`", or "`Transfer(to address, from address)`".

This of course doesn't guarantee you know what is happening. For example "`src`" is clearly
"source", but "`wad`" leaves it to the imagination.

Let us integrate this information into the next example script.

```sh
cargo run --example user_3_decode_via_apis
```
Reveals:
```json
(sample index data) Address 0x846be97d3bf1e3865f3caf55d749864d39e54cb9 appeared in 2 transactions
Examining the first transaction  (Tx 0x1a8d94dda1694bad33384215bb3dc0a56652b7069c71d2b1afed35b24c9b54df) using local node. 5 logs found.

Log 0, associated with contract: 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
        Topic Some("Deposit(address,uint256)"), signature 0xe1fffcc4923d04b559f4d29a8bfc6cda04eb5b0d3c460751c2402c5c5cc9109c decoded using 4byte.directory
        Contract ABI was obtained from Sourcify:
                Contract: {"WETH9.sol":"WETH9"}
                ...
        "event" null "Approval".
                Inputs: [{"indexed":true,"name":"src","type":"address"},{"indexed":true,"name":"guy","type":"address"},{"indexed":false,"name":"wad","type":"uint256"}]
                Outputs: null
        "event" null "Transfer".
                Inputs: [{"indexed":true,"name":"src","type":"address"},{"indexed":true,"name":"dst","type":"address"},{"indexed":false,"name":"wad","type":"uint256"}]
                Outputs: null
bingo-->"event" null "Deposit".
                Inputs: [{"indexed":true,"name":"dst","type":"address"},{"indexed":false,"name":"wad","type":"uint256"}]
                Outputs: null
        "event" null "Withdrawal".
                Inputs: [{"indexed":true,"name":"src","type":"address"},{"indexed":false,"name":"wad","type":"uint256"}]
                Outputs: null
Log 1, associated with contract: 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
        Topic Some("Transfer(address,address,uint256)"), signature 0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef decoded using 4byte.directory
        Contract ABI was obtained from Sourcify:
                Contract: {"WETH9.sol":"WETH9"}
        ... (ABI same as above)
Log 2, associated with contract: 0x106d3c66d22d2dd0446df23d7f5960752994d600
        Topic Some("Transfer(address,address,uint256)"), signature 0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef decoded using 4byte.directory
        An IPFS CID for contract metadata was in bytecode metadata: "QmZwxURkw5nD5ZCnrhqLdDFG1G52JYKXoXhvvQV2e6cmMH"
No matches for ABI were found for address: 0x106d…d600
Log 3, associated with contract: 0x1636a5dfcf7a21945c06d1bea40b52ce975ea614
        Topic Some("Sync(uint112,uint112)"), signature 0x1c411e9a96e071241c2f21f7726b17ae89e3cab4c78be50e062b03a9fffbbad1 decoded using 4byte.directory
No matches for ABI were found for address: 0x1636…a614
Log 4, associated with contract: 0x1636a5dfcf7a21945c06d1bea40b52ce975ea614
        Topic Some("Swap(address,uint256,uint256,uint256,uint256,address)"), signature 0xd78ad95fa46c994b6551d0da85fc275fe613ce37657fb8d5e3d130840159d822 decoded using 4byte.directory
No matches for ABI were found for address: 0x1636…a614
```
Note that this information also provides an avenue to make new interactions
(view methods, or new transactions).

So the ABI gives us the **bold** information:

- The user had 2 transactions total.
- One was examined more closely. It caused 5 event logs:
    - In the **WETH-9** contract
        - Deposit
            - Previously: Deposit(address,uint256)
            - Now: **Deposit(address dst, uint256 wad)**
        - A Transfer
            - Previously: Transfer(address,address,uint256)
            - Now: **Transfer(address src , address dst, uint256 wad)**
    - In a contracts not known to Sourcify (more on this below)
        - A Transfer
    - In a contracts not known to Sourcify (more on this below)
        - A Sync
        - A Swap


## A Bool

Now that we have the receipt logs and the contract abi, for every log we can recover some
lost information. Was an element in the event:

- indexed?
- not indexed?

Recall that the receipt has topics and data. This is the `Deposit(address,uint256)` log
we looked at earlier:
```json
...
topics: [
    0xe1fffcc4923d04b559f4d29a8bfc6cda04eb5b0d3c460751c2402c5c5cc9109c,
    0x0000000000000000000000007a250d5630b4cf539739df2c5dacb4c659f2488d,
],
data: Bytes(
    [
        0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0,  0, 1, 241, 97, 66, 28, 142, 0, 0
    ],
),
...
```
So `topic[0]` `0xe1fffc...` is the signature for `Deposit(address,uint256)`. But what about the second
topic and the data bytes? Which is correct of the following:

- `topic[1]` the deposit address, and the `data` the uint256?
- `data` the deposit address, and the `topic[1]` the uint256?

We fortunately can decode what is happening by looking at the ABI and checking out the
section that describes the components for this `Deposit` event:

```json
[
    {"indexed":true,"name":"dst","type":"address"},
    {"indexed":false,"name":"wad","type":"uint256"}
]
```

If something is indexed, it will go in logs, otherwise it goes in data.

So we now know that

```json
...
topics: [
    0xe1fffcc4923d04b559f4d29a8bfc6cda04eb5b0d3c460751c2402c5c5cc9109c,
        is "Deposit(address,uint256)"
    0x0000000000000000000000007a250d5630b4cf539739df2c5dacb4c659f2488d,
        is "dst" address
],
data: Bytes(
    [
        0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0,  0, 1, 241, 97, 66, 28, 142, 0, 0
    ], is "wad" uint256
),
...
```
That is helpful, receipt plus the ABI equals the ability to understand everything that a
logged event is doing.
We can later use this to create a simplified human-readable summary for a user.

## A MatchMismatch

In the above example there were two contracts that returned a 404 from Sourcify.
```
0x106d3c66d22d2dd0446df23d7f5960752994d600
```
Etherscan notes that this is a token called "Labra", and is fully verified with Sourcify.
```
0x1636a5dfcf7a21945c06d1bea40b52ce975ea614
```
Etherscan notes the bytecode matches another contract, which is fully verified as "Uniswap_v2".

Now, we could look at how these errors, happen. However, there is a much more interesting
direction to take this I think.

To find a solution that means users can obtain these data without API calls directly to
the Sourcify and 4Byte websites.

## A Datum

Now recall that the transaction receipt also gave us more than topic signatures. For
all `"indexed": true` events above, the values are included in log topics.

This means that the generic terms, like `address` and `uint256` can be replaced with the
values that we obtain from the transaction receip (starting from Topic index 1, as 0 was
the event signature).

Moreover, any address may be interrogated with `eth_getCode` again, to fetch its metadata, and
subsequently human-relatable information embedded within (such as contract name). What
this means is that you can get a tangible name for a contract, like `WETH`, and
see that `Transfer()` was the event in the `WETH` contract.

```
Examining address 0x846b...4cb9
Obtained relevent part (0x84) of the address-appearance-index (350MB)

Two transactions identified.

Using portal node to examine first transaction (0x846b...4cb9)

5 Events occurred across 3 contracts:
    - WETH Deposit(UniswapV2-router src, uint256 wad)
        - Topic index 1: (src) 0x0000000000000000000000007a250d5630b4cf539739df2c5dacb4c659f2488d (UniswapV2 Router)
        - Plus data bytes
    - WETH Transfer(UniswapV2-router src, UniswapV2-pair dst, uint256 wad)
        - Topic index 1: (src) 0x0000000000000000000000007a250d5630b4cf539739df2c5dacb4c659f2488d (UniswapV2 Router)
        - Topic index 2: (dst) 0x0000000000000000000000001636a5dfcf7a21945c06d1bea40b52ce975ea614 (UniswapV2 Pair)
        - Plus some data bytes
    - LABRA Transfer(UniswapV2-pair sender, address recipient, uint256 TransferAmount)
        - Topic index 1: (sender) 0x0000000000000000000000001636a5dfcf7a21945c06d1bea40b52ce975ea614 (UniswapV2 Pair)
        - Topic index 2: (recipient) 0x000000000000000000000000846be97d3bf1e3865f3caf55d749864d39e54cb9 (user wallet)
    - UniswapV2 Sync(uint112 reserve0, uint112 reserve1)
        - ...
    - UniswapV2 Swap(address sender, uint256 amount0In, uint256 amount1In, amount0Out, uint256 amount1Out, address to)
        - ...
```
Hopefully it is clear that with the following theoretical local toolset the
information is sufficient to be useful:

- address-appearance-index database chapters
- 4byte database chapters
- Sourcify database chapters
- Portal node

## A PrimitiveUI

We can arrive, at last, to the following:

User: Clicks on their wallet

```
Address 0x846b...4cb9 (your-wallet)

Has had 2 transactions

Transaction 1
    - WETH Deposit
        - to UniswapV2-router
        - wad 140000000000000000 (0.14 tokens)
    - WETH Transfer
        - from UniswapV2-router
        - to UniswapV2-pair
        - wad 140000000000000000 (0.14 tokens)
    - LABRA Transfer
        - from UniswapV2-pair
        - to your-wallet
        - amount 55479990315601131228 (55479990315 tokens)
    - UniswapV2 Sync
        - reserve0 27434359272513446845001
        - reserve1 67780564455540887653
    - UniswapV2 Swap
        - sender UniswapV2-router
        - amount0In 31062407892934044
        - amount1In 140000000000000000
        - amount0Out 56612235015919521660
        - amount1Out 0
        - to your-wallet

Transaction 2
    - ...
        - some loan
        - a multisig approval
        - a NFT acquisition
        - ...
```
Where the bracket values would come from a call to `decimals()` method in the contract if present.

Now we must honestly appraise whether this is a desirable goal. For it is quite a bit
of fluffing around to get here.

## A Freshening

So a user with amnesia looks at the above transaction may want to learn more.

They could be presented with present a selection of `read()` functions pulled from the
contract ABI that they can then use against their portal node.

For example, they could click:

```
Labra -> balanceOf(their-address)
> 88893921558281585194  (88893921558 tokens)
```

So now they know real, actionable information:

- They previously had a transaction where they gained:
    - `55_479_990_315` (55 billion) tokens
- They currently have:
    - `88_893_921_558` (88 billion) tokens


# Part VI - Acquisitions: Can useful remote resources become distributed (Yes, by generalising a format)

- Hypothesis: Remote 4byte and Sourcify databases can be distributed by generalising the
principles used in the Unchained Index and address-appaearance-index.
- Methods: Create minimum database architecture and publishing requirements that can be followed
to transform a centralised database in to a decentralised one.
- Conclusion: The TODD and GAMB ERCs provide a basic procedure that plausibly are a pathway
for 4byte, Sourcify and other databases to be distributed.

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

Realistic transitions:
```
Begin -> Provide supports -> Gather community -> Reduce supports -> Community adds support
```
Idealistic transitions:
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


# Part VII - Coalesce: Is a local, lighthweight personal history browser a good idea? (Plausibly so)

- Hypothesis: A personal wallet history browser with low hard disk footprint is a
worthwhile goal.
- Methods: Review and weigh the pros and cons of implementing the system as described.
- Conclusion: Significant amount of work, but plausible that the endeavour aligns with
goals of the community enough to warrant exploration.

## Useful?

I think that at the very least, the gist/flavour of the transaction is clear from this example.
We have done no special curating or nicely worded templates.
The fact that it allows you to know which contracts were involved with
your address and present `read()` functions to you that you can click on to learn more about.

It is certainly less useful that a block explorer. However, if you decided not to use a block
explorer, then it is more useful than having nothing.

## Robust?

So this is a generally applicable tool, which means it can be used on wacky,
wild unpredictable contracts.

There are some odd transactions that appear, and it may
not always be clear to a user what has happened. For example, some transactions are
"advertising" transactions that transfer a token to hundreds of reputable individuals.
A user looking at the events in such a transaction might be a little confused. Yet,
seems reasonable to believe that a clear interface presenting neutrally decoded
information could be understandable ("oh this transaction is sending x XYZ to 300 people
that seems strange and suspiciously irrelevant for me.").

## Important?

How important is it that users are able to access their own information? This seems like such
a trite example - being able to "remember your own token swap".

Yet, this also covers transactions that a user never initiated. This is a discovery tool.
What if block explorers start hiding things from view that are labelled (correctly or
incorrectly) as spam, misleading, scam, dangerous, honeypot, unsavoury or unregulated?
How would you detect that such a thing had happened? What if mistakes had been made? What
if you are in a different jurisdiction or have different preferences than the explorer.

I think it is valuable to be able to sift through raw data locally.

## Novel?

We have various ways of accessing node data. These require large hard drives though.
A great way to inspect your own history
is to roll [Otterscan](#otterscan) locally. That has the juice to be able to trace transactions
and provide all this information, but you need about 2TB all up to do so.

Our example shows how it might be possible to start living and breathing in the
future where "history expires". Expires does not mean "disappears" it means "it is distributed",
and "you need to find another way to get it". That "way" is from peers who hold
parts of the whole.

Is there a role for users to not worry about indexing and decoding databases? For example
if portal nodes or novel new clients like helios https://github.com/a16z/helios
exist, a user uses proofs to verify the correctness of any returned queries.
This means that a third party could provide "all their transaction history" without an
opportunity to give bad information. The trouble is, that this would cause 1) a reliance
on this third party (who may inconveniently disappear) 2) leaking of lots of private
data to this third party (who may sell this or retain a large database of wallets-entity mappings
that may be breached and leaked.

So it does seem like the presence of distributed databases (indices, signatures, abis)
does provide a novel contribution for a "users own their data" future.

## Hard?

Everyone needs to get and serve data by using software akin to torrent clients. You
get it because it gets the things, and in the background it keeps the network zippy.

Dedicated servers can be stood up to start off. Then users start to come along.
If they have to adjust settings to stop sharing data, then many will not, and the
network will thrive.

Does it make sense to "shard everything" like this?

I have taken a look at Sourcify and 4byte. Is the
[Time Ordered Distributable Database](#erc-time-ordered-distributable-database)
the right model for these? If so, where does it make sense to implement and embed them?
Does it make sense to shoehorn different databases into this common schema, or is
this adding friction to already under-resourced and under-appreciated goods?

## Untouched?

Other related directions/extensions:

- Repeadedly fetching information from a portal node until tracing a single transaction is
possible (I do not think it would add much to a wallet user's experince above what can be
had from receipts)
- Preparing a user to make new transactions by polling the portal node for current
contract state. Once a user has their history, that information
can inform what transactions they might want to make (get more/less of token they had
forgotten about). Perhaps connecting portal nodes to decentralised front ends is a cool idea.
The transaction history could trigger a search for decentralized front ends for contracts
previously interacted with. (You previously swapped RAI on uniswap, here are IPFS front
ends for Uniswap and Rai manager in case you want them).
- Using this sort of information to feed some sort of otterscan-lite. This could
provide a familiar interface to the data fetched/presented using the methods in this post.
- Integrating address-appearance-index (or 4byte/sourcify) as overlay networks the portal network.
If nodes are already maintaining partial databases using distance based metrics and node ids,
you might be able to have nodes automatically host "their regions" of these other databases.
This could be an extension of the portal spec, and implementation in [Trin](#trin).

## Next?

In conclusion, any database that you want to distribute among users can be:

- Divided according to
[EIP-Timd-Ordered-Ditributed-Database](#erc-time-ordered-distributable-database)
- Broadcast using the already-deployed
[EIP-Generalized-Attributable-Manifest-Broadcaster](#erc-generic-attributable-manifest-broadcaster)

This could be all three of:
- Address-appearance-index
- 4byte directory
- Sourcify

Which together provide a means for a regular user with a small computer to perform basic
accounting such as reviewing past actions and checking the current state of their wallet affairs.

There could be other databases amenable to such a concept too.

---

## Thanks :)

I'd love to hear any thoughts you have on things! I'm @eth_worm on twitter and @perama-v on github.

---

## References

### Ethereum

A secure decentralised transaction ledger.

https://ethereum.github.io/yellowpaper/paper.pdf

### Ethereum consensus specification

Ethereum proof-of-stake specifications.

https://github.com/ethereum/consensus-specs

### EIP-4444

Bound Historical Data in Execution Clients.

https://eips.ethereum.org/EIPS/eip-4444

### Portal Network

Lightweight protocol access by resource constrained devices.

https://github.com/ethereum/portal-network-specs/blob/master/README.md

### trueblocks-core

TrueBlocks creates an index that lets you access the entire Ethereum chain directly from your local machine.

https://github.com/TrueBlocks/trueblocks-core

### Unchained Index

The Unchained Index is a naturally-sharded, easily-shared, reproducible, and minimally-sized.
immutable index for EVM-based blockchains.

https://trueblocks.io/papers/2022/file-format-spec-v0.40.0-beta.pdf

### SSZ Spec

Simple Serialize (SSZ) is a standard for the encoding and merkleization of structured data.

https://github.com/ethereum/consensus-specs/blob/dev/ssz/simple-serialize.md

### Snappy

Consensus spec: SSZ-snappy encoding strategy.

https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/p2p-interface.md#ssz-snappy-encoding-strategy

Google: Snappy, a fast compressor/decompressor.

https://github.com/google/snappy

### Lighthouse

An Ethereum consensus client in Rust.

https://github.com/sigp/lighthouse

### Beacon chain

The specification for Phase 0 -- The Beacon Chain.

https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/beacon-chain.md

### Trin

Trin is a Rust implementation of a Portal Network client.

https://github.com/ethereum/trin

### Sourcify

Sourcify enables transparent and human-readable smart contract interactions through automated Solidity contract verification, contract metadata, and NatSpec comments.

https://sourcify.dev/

### IPFS CID

Self-describing content-addressed identifiers for distributed systems.

https://github.com/multiformats/cid

### 4Byte directory

List of 4byte identifiers for EVM smart contract functions

https://www.4byte.directory/

https://github.com/ethereum-lists/4bytes

### Otterscan

A blazingly fast, local, Ethereum block explorer built on top of Erigon

https://github.com/wmitsuda/otterscan

### Erigon

Ethereum implementation on the efficiency frontier

https://github.com/ledgerwatch/erigon

### ERC-time-ordered-distributable-database

A format for useful peer-to-peer databases.

https://github.com/perama-v/TODD

### ERC-generic-attributable-manifest-broadcaster

A contract for announcing newly published metadata.

https://github.com/perama-v/GAMB

### address-appearance-index-specs

A specification of an index that maps addresses to the transactions that they appear in.

https://github.com/perama-v/address-appearance-index-specs

### Min-know

An implementation of the address-appearance-index-specs.

https://github.com/perama-v/min-know

