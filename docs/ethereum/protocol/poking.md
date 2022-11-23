---
layout: page
title:  "Protocol poking"
permalink: /ethereum/protocol/poking
toc: true
---
```
Abstractions in pen;
protocols for protocols.
Recipes for glue
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

Really it started with this kernel of an idea:
```sh
Introspection on ones own wallet is not really possible without a bit
of trust-hand-waving or a trusty dedicated ubuntu machine.

But it could be! The ingredients seem to be there at least.
```

> prōtos "first" + kolla "glue"

---

- [Part I - Musing: Is there an underserved user in the history-expiry roadmap? (Yes, a regular wallet user)](poking/part_1.md)
- [Part II - Code: Can you divide a useful index into tiny useful parts? (Yes, prototype)](poking/part_2.md)
- [Part III - Specifying: Could other clients use a divided index? (Yes, with a spec)](poking/part_3.md)
- [Part IV - Examining: Does a divided index result in anything useful? (Yes, personal history)](poking/part_4.md)
- [Part V - Remotes: Can remote resources make tx history human-readable? (Yes, through decoding and ABIs)](poking/part_5.md)
- [Part VI - Acquisitions: Can useful remote resources become distributed (Yes, by generalising a format)](poking/part_6.md)
- [Part VII - Coalesce: Is a local, lighthweight personal history browser a good idea? (Plausibly so)](poking/part_7.md)
- [Part VII - Hoovering: Can private API-based database curators coexist with distributed database design? (Yes, by standards for distribution)](poking/part_7.md)

---


If you like, these are the spoilers that came out of this:

- [An index spec to discover your own transactions with minimal data](https://github.com/perama-v/address-appearance-index-specs)
- [An implementation prototype of the spec to build the index](https://github.com/perama-v/min-know)
- [A generalisation of the format of the index as a potential ERC standard](https://github.com/perama-v/TODD)
- [A generalisation of the broadcast mechanism as a potential ERC standard](https://github.com/perama-v/GAMB)


The "final product" is this "user interface" demo:

```
[User: Clicks on their wallet]

Address 0x846b...4cb9 (imagine this is "your-wallet")

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

It's a little plain, but it is also evident what "transaction 1"
is roughly about. This interface is possible in a resource
constrained device in a post-EIP-4444 world. Where the full
decentralised local mini-account explorer
for a user could be <1GB.

Using theoretical combination of:
- Portal network client
- Pieces of special distributable databases:
    - An index of address appearances
    - An event signature database
    - A contract ABI database
- A broadcasting system to get the manifests of:
    - The index
    - The event signature database
    - The contract ABI database

The how-s and the why-s are covered in post-parts-1-to-7.
