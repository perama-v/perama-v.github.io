---
layout: page
title:  "Protocol poking"
permalink: /ethereum/protocol/poking/part_7
toc: false
---


# Part VII - Coalesce

- Hypothesis: A personal wallet history browser with low hard disk footprint is a
worthwhile goal.
- Methods: Review and weigh the pros and cons of implementing the system as described.
- Conclusion: Significant amount of work, but plausible that the endeavour aligns with
goals of the community enough to warrant exploration.

(Continued from [poking part 6](part_6.md))

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
if portal nodes or novel new clients like [Helios](#helios) exist, a user uses proofs to verify the correctness of any returned queries.
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
[EIP-Time-Ordered-Ditributed-Database](#erc-time-ordered-distributable-database)
- Broadcast using the already-deployed
[EIP-Generalized-Attributable-Manifest-Broadcaster](#erc-generic-attributable-manifest-broadcaster)

This could be all three of:
- Address-appearance-index
- 4byte directory
- Sourcify

Which together provide a means for a regular user with a small computer to perform basic
accounting such as reviewing past actions and checking the current state of their wallet affairs.

There could be other databases amenable to such a concept too.

Continue on to [poking part 8](part_8.md)

---

## Thanks :)

I'd love to hear any thoughts you have on things! I'm @eth_worm on twitter and @perama-v on github.

---

## References

### Helios

A fast, secure, and portable light client for Ethereum

[https://github.com/a16z/helios](https://github.com/a16z/helios)

### Trin

Trin is a Rust implementation of a Portal Network client.

[https://github.com/ethereum/trin](https://github.com/ethereum/trin)

### Otterscan

A blazingly fast, local, Ethereum block explorer built on top of Erigon

[https://github.com/wmitsuda/otterscan](https://github.com/wmitsuda/otterscan)

### ERC-time-ordered-distributable-database

A format for useful peer-to-peer databases.

[https://github.com/perama-v/TODD](https://github.com/perama-v/TODD)

### ERC-generic-attributable-manifest-broadcaster

A contract for announcing newly published metadata.

[https://github.com/perama-v/GAMB](https://github.com/perama-v/GAMB)
