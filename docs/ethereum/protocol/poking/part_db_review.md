
## The Transformer

A local tool that lets a user participate in creating a distributable database.

Any DB in -> TODD out

TODD + new data -> Updated TODD out

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

## The Survey

Categories:
- Signatures
- Names
- Tags
- ABIs
- Appearances

| Database | Keys | Values | Format | Data open source | Growth mechanism | Distribution | Size | Link |
|-|-|-|-|-|-|-|-|-|
|Sourcify|Contract Address|ABI|JSON|Open source|Web UI, API|Free API, scrape, IPNS, github clone |16GB|[sourcify.dev](https://sourcify.dev/)|
|4Byte site|Hex signature|Text signature|.txt files|Closed source (but published to GH)|Web UI, API|Free API|962,000 entries|[4byte.directory](https://www.4byte.directory/)|
|4byte github|Hex signature|Text signature|.txt files|Open source|Periodic PR using data from 4byte.directory|Clone github|-|[github](https://github.com/ethereum-lists/4bytes)|
|Ethereum tags|Address|Tags|-|Closed source|-|-|Free API|[samczsun.com](https://tags.eth.samczsun.com/)|
|Topic0|Hex event signature|Text event signature|-|-|-|-|-|[github](https://github.com/wmitsuda/topic0)|
|etk-4Byte|Hex signature|Text signature|Hard coded rust crate|-|-|-|-|[crates.io](https://crates.io/crates/etk-4byte)|
|Trueblocks Names|Address|Name|-|Yes|-|-|-|[trueblocks.io](https://trueblocks.io/docs/using/introducing-chifra/)|
|Kleros names|Address|Tas|-|Open source|Token Curated Registry|-|-|[kleros.io](https://curate.kleros.io/tcr/100/0x76944a2678A0954A610096Ee78E8CEB8d46d5922)|
|RolodETH|Address|Name, tags|JSON files|Open source|GH PR|GH Clone|640,437 entries|[github](https://github.com/verynifty/RolodETH)|
|Scraped Etherscan labels|Address|Name, tags|JSON file|Open Source|GH PR|Clone|3.39MB|[github](https://github.com/brianleect/etherscan-labels)|
|Unchained Index|Address|Appearances (tx ids)|binary|Open source|Scrape node, process, pin|IPFS via smart contract publisher|80GB|[trueblocks.io](https://trueblocks.io/docs/install/get-the-index/)|
|-|-|-|-|-|-|-|-|-|
|-|-|-|-|-|-|-|-|-|
|-|-|-|-|-|-|-|-|-|


## Questions

- Are DB sizes large enough to warrant dividing the DB (volumes only vs chapters)?
- Is there a neat reconciliation processes for the different databases?
    - E.g., For two divergent dbs, (TODD_1, TODD_2) is there reconcile process
    to make TODD_3 contain both TODD_1 and TODD_2. E.g., merge tips, or scrape and build.
