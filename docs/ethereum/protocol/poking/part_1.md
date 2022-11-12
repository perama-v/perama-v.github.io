---
layout: page
title:  "Protocol poking"
permalink: /ethereum/protocol/poking/part_1
toc: false
---


# Part I - Musing

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
have shared partial nodes (the excellent [portal network](#portal-network)), it doesn't advance you in the quest
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
gradually more likely to implement history expiry ([EIP-4444](#eip-4444))
as part of The Purge. This allows nodes to be teeny-weeny, which is nice for users.

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


Continue on to [poking part 2](part_2.md)

----
## References

### Ethereum

A secure decentralised transaction ledger.

https://ethereum.github.io/yellowpaper/paper.pdf

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

### Erigon

Ethereum implementation on the efficiency frontier

https://github.com/ledgerwatch/erigon

### address-appearance-index-specs

A specification of an index that maps addresses to the transactions that they appear in.

https://github.com/perama-v/address-appearance-index-specs
