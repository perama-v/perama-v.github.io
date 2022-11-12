---
layout: page
title:  "Protocol poking"
permalink: /ethereum/protocol/poking/part_2
toc: false
---


# Part II - Code

- Hypothesis: The Unchained Index can be divided into tiny functional parts.
- Methods: Createa a rust library that transforms the index into a new format.
- Conclusion: Dividing addresses by first two hex characters results in <500MB parts.

(Continued from [poking part 1](part_1.md))

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

Continue on to [poking part 3](part_3.md)

----
## References
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
