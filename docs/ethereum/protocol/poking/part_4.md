---
layout: page
title:  "Protocol poking"
permalink: /ethereum/protocol/poking/part_4
toc: false
---

# Part IV - Examining

- Hypothesis: The information is useful for a regular user.
- Methods: Use local Ethereum node as if it were a Portal Node, use the index to get transaction receipts relevant to the user.
- Conclusion: The receipts for an address are obtainable but not immediately useful without
additional work.

(Continued from [pokinng part 3](part_3.md))

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

Continue on to [poking part 5](part_5.md)

----
## References

### Ethereum

### Portal Network

Lightweight protocol access by resource constrained devices.

[https://github.com/ethereum/portal-network-specs/blob/master/README.md](https://github.com/ethereum/portal-network-specs/blob/master/README.md)

### Trin

Trin is a Rust implementation of a Portal Network client.

[https://github.com/ethereum/trin](https://github.com/ethereum/trin)

### Sourcify

Sourcify enables transparent and human-readable smart contract interactions through automated Solidity contract verification, contract metadata, and NatSpec comments.

[https://sourcify.dev/](https://sourcify.dev/)

### IPFS CID

Self-describing content-addressed identifiers for distributed systems.

[https://github.com/multiformats/cid](https://github.com/multiformats/cid)

### Min-know

An implementation of the address-appearance-index-specs.

[https://github.com/perama-v/min-know](https://github.com/perama-v/min-know)

