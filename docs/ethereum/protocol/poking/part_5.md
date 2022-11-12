---
layout: page
title:  "Protocol poking"
permalink: /ethereum/protocol/poking/part_5
toc: false
---


# Part V - Remotes

- Hypothesis: Additional remote databases can make retrieved history useful for a normal user.
- Methods: For retrieved transaction receipts, call external 4byte and sourcify APIs and decode data.
- Conclusion: 4byte and sourcify provide sufficient data for a user to understand their wallet activity.

(Continued from [poking part 4](part_4.md))

## A LocalFirst

So far we have only used information from our portal node and the index.

Next we need more information. This can go two ways:

- APIs (unhealthy)
- Corralling the data locally (healthy)

We will start with an API as a test, and then find a better way.

## A FourByte

Now we need an event signature database. People have been dutifully computing and collecting
these. So later we could give the database to everyone (TBD DB size). For
now we skip that and look it up in [4byte directory](#4byte-directory)

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

Continue on to [poking part 6](part_6.md)

----
## References

### 4Byte directory

List of 4byte identifiers for EVM smart contract functions

https://www.4byte.directory/

https://github.com/ethereum-lists/4bytes
