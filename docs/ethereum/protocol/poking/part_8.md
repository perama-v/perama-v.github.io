---
layout: page
title:  "Protocol poking"
permalink: /ethereum/protocol/poking/part_8
toc: false
---

# Part VIII - The Hoovers

- Hypothesis: Siloed databases duplicate efforts and reduce coordination.
- Methods: Enumerate databases and examine dependency properties and procedures.
- Conclusion: Database providers effectively curate important data but having closed
and non-standardised backends prevents coordination.

(Continued from [poking part 7](part_7.md))

## The Survey

Let us examine some important domains to see what databases are present:

Categories:
- ABIs
- Signatures
- Names and Tags
- Appearances

### ABIs

What is the interface for this deployed contract?

| Database | Keys | Values | Format | Data open source | Growth mechanism | Distribution | Size | Link |
|-|-|-|-|-|-|-|-|-|
|Sourcify|Contract Address|ABI|JSON|Open source|Web UI, API|Free API, scrape, IPNS, github clone |16GB|[sourcify.dev](https://sourcify.dev/)|
|Etherscan|Contract Address|ABI|JSON|Closed Source|Free API|Free API|-|[etherscan.io](https://docs.etherscan.io/api-endpoints/contracts)|

### Signatures

What is the human readable version of this method or event?

| Database | Keys | Values | Format | Data open source | Growth mechanism | Distribution | Size | Link |
|-|-|-|-|-|-|-|-|-|
|Ethereum tags|Address|Text signature|JSON|Closed source|Free API|Free API|-|[samczsun.com](https://sig.eth.samczsun.com/reference)|
|4Byte site|Hex signature|Text signature|.txt files|Closed source (but published to GH)|Web UI, API|Free API|962_000 entries|[4byte.directory](https://www.4byte.directory/)|
|4byte github|Hex signature|Text signature|.txt files|Open source|Periodic PR using data from 4byte.directory|Clone github|962_000 entries|[github](https://github.com/ethereum-lists/4bytes)|
|Topic0|Hex event signature|Text event signature|.txt files|Open source|Clone and parse Sourcify registry|GH Clone|7_800 entries|[github](https://github.com/wmitsuda/topic0)|
|etk-4Byte|Hex signature|Text signature|Hard coded rust crate|Open source|Clone 4Byte directory|GH Clone|-|[crates.io](https://crates.io/crates/etk-4byte)|

### Tags and Names

What is the meaningful label for this address?

| Database | Keys | Values | Format | Data open source | Growth mechanism | Distribution | Size | Link |
|-|-|-|-|-|-|-|-|-|
|Ethereum tags|Address|Tags|JSON|Closed source|Free API|Free API|-|[samczsun.com](https://tags.eth.samczsun.com/)|
|Trueblocks Names|Address|Name, Tags|custom binary file|Yes|Trueblocks-driven|Install trueblocks-core|9.2MB|[trueblocks.io](https://trueblocks.io/docs/using/introducing-chifra/)|
|Kleros names|Address|Tags|-|Open source|Token Curated Registry|-|-|[kleros.io](https://curate.kleros.io/tcr/100/0x76944a2678A0954A610096Ee78E8CEB8d46d5922)|
|RolodETH|Address|Name, tags|JSON files|Open source|GH PR|GH Clone|640_437 entries|[github](https://github.com/verynifty/RolodETH)|
|Scraped Etherscan labels|Address|Name, tags|JSON file|Open Source|GH PR|Clone|3.39MB|[github](https://github.com/brianleect/etherscan-labels)|

### Address Appearances

Where did my address appear?

| Database | Keys | Values | Format | Data open source | Growth mechanism | Distribution | Size | Link |
|-|-|-|-|-|-|-|-|-|
|Unchained Index|Address|Appearances (tx ids)|custom binary files|Open source|Create locally and share|IPFS via smart contract publisher|80GB|[trueblocks.io](https://trueblocks.io/docs/install/get-the-index/)|
|Etherscan|Address|Appearances (txs)|JSON|Closed source|Free API|Free API|-|[etherscan.io](https://docs.etherscan.io/api-endpoints/accounts)|
|(Gedankenexperiment) address-appearance-index*|Address|Appearances (tx ids)|SSZ files|Open source|parse Unchained Index locally and share|IPFS via smart contract publisher|80GB|[github spec](https://github.com/perama-v/address-appearance-index-specs)|

Note: address-appearance-index is a prototype/conecept only. As seen in the table it is a derivative
of the Unchained Index and thus duplicates effort to get the same data in a new format.
I designed the index as the starting point in an exploration of distributable databases.
It may be a wiser approach to lean more heavily into Unchained Index than try for a
slightly different variation/derivative of it.

## The Embedding

When a database is duplicated, the existing database infrastructure is left to waste.

For example, here is an example where the public 4byte registry is absorbed into a closed
source database with a free API. The API is then embedded in an open source tool because
it has better uptime characteristics.

https://github.com/foundry-rs/foundry/issues/1672

Now there are two competing entities trying to solve the same problem,
in a way that detracts from the other - rather than strengthening it. This creates
short term solutions that ultimately fail our decentralised needs: Trust me, I'll keep
maintaining this open API for free forever. It may be true, but it makes the open source
tool vulnerable.

What if the API is shut down? Well you could try to return to the original sources of the
hoovered data. But they may no longer exist because the customers drawn to the "more reliable"
closed API and the effort was shut down.

Would it not be better if:

- The closed API that gains new data publishes it in a way that contributes to the original data?
- A Foundry user had the option of pinning parts of the database?

Agreeing on distributable database formats does not mean "everything has to be slow". It means
that:

- Users can contribute to hosting the database.
- Separate providers can contribute to the database.

## The Formula

Databases can be redesigned so that all the above duplicated efforts could point to the
same underlying content.

These entities:

- Samczsun's Ethereum signatures
- Snake charmers' 4Byte
- Williams' Topic0
- Quilt's etk-4Byte

Could all point to content identifiers (CIDs) for their databases. When they update
their local database, they could use manifests to coordinate without communicating:

- Go to a manifest publishing contract
- Use the term "fourbyte" to get a list of IPNS names
- Check each IPNS to get a manifest
- Look at what each manifest contains. If another manifest has more data,
use the CIDs to get that data.
- Now if you have additional data, create new additions with their own
CIDs and preoduce a new mainfest.
- Publish your manifest under your INPS name.

Thus, separate parties could build the same database in a distributed way. They
can provide the data via a free and performant API, but a user can also just
use the manifests to get the latest data. Users can also pin.

## The Formats

Each database type could have an agreed upon format for the values in key-value pairs.

This is a public declaration / specification where one states:

- A JSON format
- A plain string
- An SSZ container

Interested parties could all agree that the format for a key-value pair is the following json:

```json
{
    "key": "hex_xyz",
    "value": "readable_text_xyz"
}
```
## The Sealing

A database that seals old data and never touches it can share pieces with peers.

This is a public declaration / specification where one states the rules to decide how
to group new key-value pairs:

- When a key-value pair does not fit in the current Volume, an must go in a new Volume
- Which Chapter the the key-value pair fits in

For example someone could publish a specification for fourbyte data:

- Fourbyte Volumes MUST contain EXACTLY 1_000 entries
- Fourbyte Chapters contain entries whose keys all start with the same hex character.

So if you had 1_200 entries to add to the database, you would know that you can
only seal 1_000 of them into a Volume. Additionally, the volumes you publish will
have 16 different chapters (0x0... to 0xf...).

So if 4byte.directory currently has 961,200 entries they know that at 962_000 they
will publish a manifest containing all existing CIDs plus:
```
- volume 961_000
- volume 962_000 (4byte.directory's latest addition)
    - chapter 0x0 CID xyz
    - chapter 0x1 CID xyz
    - ...
    - chapter 0xf CID xyz
```

That way, other database maintainers can see your published update and build the
next volume on top of your latest volume.

If Samczsun incorporates 962_000 and has an additional 1_000 entries, they
could publish a manifest containing 962_000 and 963_000

```
- volume 962_000 (first published at 4byte.directory's IPNS)
- volume 963_000 (published at Samczsun's IPNS)
    - chapter 0x0 CID xyz
    - chapter 0x1 CID xyz
    - ...
    - chapter 0xf CID xyz
```


## The Transformer

To make this process smooth, Transformer software could be written to
produce Volumes/Chapters and Manifests of the right format.

For example, A "TODD" format, where TODD is a Time-Ordered Distributable Database:

https://github.com/perama-v/TODD

This could exist as a local tool that lets a user participate in creating a
distributable database of any artitrary sort:

Any DB in -> TODD out

TODD + new data -> Updated TODD out

This could re-use common functions (manifest creation) and only require
a custom adapter for each database type (ABIs, signatures, tags).

## The Requirements

To be Transform-able, one needs to outilne a pre- and post-TODD format for the new
data.

E.g., Imagine you have neat TODD pieces (volumes and chunks).
Then you have a .csv list of new data you want to add. Perhaps it will become the
next 3 volumes in the database. The transformer will accept data in a specific input
format which could be different for each database (binary files, .tsv, .csv).

All that is required is that the input data have a parser that can loop over all the keys
in the data to be incorporated.

.csv -> write transformer parser -> parse -> todd

.tsv -> write transformer parser -> parse -> todd

.bin -> write transformer parser -> parse -> todd

The idea is that you want people to be able to create data in whatever format is convenient
for them. Perhaps this format is different between creators - it does not matter because
the transformer outputs TODD format which both parties agree on.

## The Contract

In a specification, say for fourbyte signatures, one would declare an arbitrary topic,
thus making it clear under which term ("fourbyte" "4byte" "event-signatures" etc.)
publishing IPNS names should be registered.

See here for more rationale on IPNS name based broadcasting:
https://github.com/perama-v/GAMB/issues/1

## The Hoovers

Many benevolent actors want to create databases as public goods. This
can result in duplication of efforts and vampire attack of users.

Databases that grow do benefit from easy to use APIs/UIs that let users contribute small pieces
of data (e.g., pubish ABI or event signature by API). So this is not a claim that
"Free API providers are the problem". Updating databases is a criticial role.

The problem as I see it is the Molochian failure to coordinate.

The solution is Schelling points for database formats and database distribution
mechanisms.

Data can be transparent, content-addressble pieces that can be periodically and permanently
shared back to the user who then acts as provider.

