---
layout: page
title:  "Prove L1 ownership on L2"
permalink: /cairo/game/ownership-proof/
toc: false
---

A Cairo contract that accepts a Merkle proof and then registers user-DOPE mapping
in the `userRegistry`.

The purpose is to allow asset holders (DOPE, PAPER) to register ownership on L2 by providing
a Merkle proof of ownership. Avoids L1 gas cost completely.

Elements:

- A snapshot of L1 DOPE owners is made at some L1 block height
- The list of dopeID-publicKey pair entries is modified
    - A system for sorting DOPE NFTs into efficient discrete categories is made. For
    example if the game has mechanisms for probabilistic `[fights, muggings, bribes]`, each
    NFT is assigned a factor (out of 200) for each, where 100 is no effect, 80 is 0.8x.
    The factors may be based on the trait rarity.
        - E.g., NFT has rare weapon, common shoes, rare ring: their stats might be
        `[1.4, 0.7, 1.3]`
        - Or the classification might be categorical E.g., NFT has rare weapon,
        common shoes, rare ring: their stats might be high-low-high `[2, 1, 2]`.
- The user provides the Merkle proof using their Ethereum public key to sign a message
- The contract accepts the traits provided in the proof and stores them in
`userRegistry.cairo`. Games may then be spawned from this registry without needing additional
merkle proofs.

Mechanism design is a WIP. The alternative is to use a L1toL2 message.