---
layout: page
title:  "Wallet design"
permalink: /ethereum/wallet-design
toc: true
---

The Ethereum transaction fee mechanism changes with
[EIP1559](https://hackmd.io/@timbeiko/1559-updates/https%3A%2F%2Fhackmd.io%2F%40timbeiko%2F1559-resources).

Here are some thoughts on how a wallet might make the most of the opportunity to improve
how a user feels when they send a transaction.

1. There are new terms used to describe different parts of the transaction. These are
outlined below, but first, some effort should be made to *not* expose these terms by default
to a user.
2. There are new ways in which a transaction fee can be architected. Effort should be
placed on *hiding* this flexibility from the user.

## Demo UI

Here is what I imagine a wallet interface to feel like:

```
Maximum transaction fee:
x.xxx ETH (y USD).

|info|        |Send|
```

The user is sending a transaction, they see the cost, they send.

- The fee in ETH would have 3 significant figures (E.g., 0.0322 ETH or 0.100 milliETH).
    - Where ether values are small, use SI notation rather than many decimal places
(0.100 milliETH or milliether instead of 0.000100 ETH).
    - Where "Gwei" is considered (elsewhere in the UI), use nanoETH or nanoether instead.
- The USD (or local currency) would have the same (E.g., $69.8 or $0.217).
- The `info` button is outlined below.
- The `send` button sends the transaction.

Before exploring what should be included under info, consider why the above format
is not the current wallet standard. The new feature that 1559 brings is the ability
to know the *fair going price* of a transaction. This experience is delivered by leveraging
that knowledge, which is now in the header of the last block.

(implementation note, the fee is `2 * BaseFee`, which is found in the block header, and
is accessed through the feeHistory API from the
[JSON RPC spec](https://github.com/ethereum/eth1.0-specs/blob/master/json-rpc/spec.json))

## The `|info|` button

The user is not entirely satisfied with the simple interface and wants more.

```
|<- save & back|
x.xxx ETH fee is
m times the current fee.
Any excess will be refunded.

This covers n minutes of
full congestion.

-1-----------2-----------10-
             ^
|refine|              |send|
```

This informs those who are curious about what the fee gets them.

It gets them a good deal, as indicated by the term 'refund'. It
is strongly implied that the wallet is going to make sure the transaction happens
and that it is going to get a refund if possible.

The display also gives a user an expectation that there is a chance for failure,
but that they are covered to some extent. They can drag the slide to 10x the current fee,
and can see how many minutes of congestion that will cover them for, and how much that will
cost. This is a 'safe' activity, because the refund mechanism protects this aspect of the fee.

This page can be thought of as 'eventuality', users can express their strong desire for
urgent inclusion here. The time is displayed as with minutes and seconds
(E.g., 4min 2s or 0min 56s)

The `|refine|` button is exposed now, which tells the user that there is more that could be
done, but that it may be nuanced. Perhaps most people would return to the main page, or
take a quick look and then return.

(**implementation note**, `n` minutes is calculated by taking the slider `val` and
multiplying by the block time `14 * val` before converting to mins and secs. The x.xxx ETH
fee is calculated as protocol BaseFee, slider val and transaction gas
conversion: `BaseFee * val * TxGasLimit`)

## The `|refine|` button

Here the user may want to express their preference for urgency. This is expressed in terms
of changing a Gas Premium.

```
|<- save & back|        |limit & nonce|
x.xxx ETH fee includes:
m times basefee (excess refunded).

-1-----------2-----------10-
             ^

Plus a Gas Premium (always paid):
2 nanoether
Recent Gas Premium values are:

[low 1]  [median 2]  [high 5]

Or enter Gas Premium: [ 1 ] nanoether.

|Gas Premium stats|              |send|
```

Not that the low, med, high values are the 1st, 50th and 90th percentiles.
See *implementation notes* at the bottom of this section.
Now the user is exposed to the breakdown of their transaction.

They can clearly see that:
- One part is refunded.
- One part is for urgency and will not be refunded.

The user can importantly make the changes here and send their transaction
from this window, without having to return. The slider here does not
have to repeat the time estimates that the previous page showed, because they have
passed through that window already, and if need, can return.

The Gas Premium can be selected from the range presented, suiting a user who is in a hurry.
The Gas Premium is pre-selected to be `|low|` as an endorsement that this is normal behaviour.

To summarise the journey of a user 'in a rush to send an urgent transaction':

1. Open wallet
2. Click `|info|`
3. Click `|refine|`
4. Click `|high|`
5. Click `|send|`

This is an excellent user experience because the value are populated and
the path to get there is easy to find. Importantly, the non-refundable
element is not 'front page', which reduces the chance of people
paying for urgency that they do not need.

The `|limit & nonce|` is also introduced at this level, and will be
clear to those who need it what that means. It separates the transaction
composition (gas limit and nonce) from the price of the transaction. The details
of this are not described here, for they do not differ after EIP1559.

The final button is the Gas Premium stats button.

(**implementation note**, the Gas Premium is obtained by querying the feeHistory API from
the json RPC for percentile Gas Premium values. The API query will be to the most recently mined
block. In this instance, percentiles queried are: 0th, 50th and 90th percentile.
The slider calculations are outlined in the previous section.)

## The `|Gas Premium stats|` button

The `|Gas Premium stats|` button provides more information that could be actioned
by a rational and interested user. The word 'stats' implies that the
page is extra information, but not important for sending a transaction.
The stats convey the 'state of the fee market', and can visually
illustrate the 'context' in which a transaction will be competing.

The stats that this page exposes are:

- 12 most recent basefees. *What is the trend?*
- The Gas Premium values from the last block. *What is the competition?*

```
|<- back|

Base fee
of last 12 blocks.
|                  |
|                  |
|  . .             | F
|     .            | E
|       . .     .. | E
|          .....   |
  Older       Newer

Gas Premium distribution
of last block.
|                 .| %
|                . | B
|               .  | E
|       ........   | L
|    ...           | O
|....              | W
 Lowest     Highest
```

Both charts provide rich information that different users
can utilise in a number of different ways.

(**implementation note**, the baseFee is obtained from the header of the last 12 blocks, and
is obtained from the JSON RPC feeHistory API.
The feeHistory is obtained by a call to the
[feeHistory API](https://github.com/ethereum/eth1.0-specs/blob/master/json-rpc/spec.json)
requesting the percentiles
at a granularity that is enough to make a nice curve. E.g. every 5th percentile minimum perhaps
[0,5,10,15,20...,90,95,100]).

## Terminology.

Details for new components of the transaction fee are outlined below (from
[here](https://hackmd.io/@q8X_WM2nTfu6nuvAzqXiTQ/1559-wallets)).

**`baseFeePerGas`**

Generated by the protocol, recorded in each block header.
Represents the minimum `gasUsed` multiplier required for a tx to be included in a block.
This is the part of the tx fee that is burnt: `baseFeePerGas * gasUsed`.

**`maxPriorityFeePerGas`**

Set by the wallet. Added to transactions, represents the part of the tx fee that goes to the miner.

**`maxFeePerGas`**

Set by the wallet. Users set this. Represents the maximum amount that a user is willing to
pay for their tx (inclusive of baseFeePerGas and maxPriorityFeePerGas).
The difference between maxFeePerGas and baseFeePerGas + maxPriorityFeePerGas
is “refunded” to the user.

Link to the JSON RPC spec for the
[FeeHistory](https://github.com/ethereum/eth1.0-specs/blob/master/json-rpc/spec.json)
API details.