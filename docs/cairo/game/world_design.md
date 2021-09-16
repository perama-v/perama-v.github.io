---
layout: page
title:  "World building"
permalink: /cairo/game/world
toc: false
---

Ethereum allows Constant Product Market Makers (CPMMs), or Automated Market Makers to flourish.
Layer 2 makes small value transactions viable. What magic may arise from the combination?

This is an exploration in to how a community driven game called Dope Wars can be built on StarkNet.

TL;DR
[play with the engine](https://voyager.online/contract/0x02c9163ce5908b12a1d547e736f8ab6f5543f6ef1fd4994c7f1b146087f3279a#writeContract)
or learn some Cairo by [reading the engine code](./game_engine_v1.md) and contribute to the repo
[here](https://github.com/dopedao/RYO).

## Background

[Dope Wars](https://dope-wars.notion.site/Home-e237166bd7e6457babc964d1724befb2) is a
is a [loot](https://etherscan.io/token/0xff9c1b15b16263c61d017ee9f65c50e4ae0113d7)-style
[contract](https://etherscan.io/token/0x8707276df042e89669d69a177d3da7dc78bd8723)
where players own a minimalistic list of items as a non-fungible (ERC-721) token (DOPE).

The inspiration for the DOPE universe stems from a long line of tersely-termed
'business simulation games'. Titles include
[Lemonade Stand](https://en.wikipedia.org/wiki/Lemonade_Stand) and
[Drug Wars](https://en.wikipedia.org/wiki/Drug_Wars_(video_game)).
These games have had smalle enough [game logic](https://gist.github.com/mattmanning/1002653)
to existed on low-capacity devices, such as graphing calculators. StarkNet is a calculator,
whose sums are checked and proven by the EVM (another calculator) and made publicly availble
for scrutiny by all.

Where calculators enabled some peer to peer gaming with cables in a classroom, StarkNet/EVM pair
allow peer to peer gaming with strangers on the internet. With no teacher to shut down the fun!

## Outline

The game is arbitrage. The goal is trade items for in-game monopoly-money and have the most money
at the end of the round. The world consists of locations with markets that
have different prices for various items. The players move between locations and make trades.
Unpredictable elements, such as theft and market-wide events create instability and arbitrage
opportunity. The mechanics are transparent and market prices are always visible, however turns
are rate-limited.

The game is interesting because it requires complex plannning, risk management and perhaps some
out-of-band social coordination. More on possible mechanics
[here](https://dope-wars.notion.site/dope-22fe2860c3e64b1687db9ba2d70b0bb5). The StarkNet
backend for the game allows for interesting outcomes, such as the winner of a round to be
issued a transferrable trophy.

## System Architecture

### Core Layer 2 (StarkNet/Cairo)
- `UserRegistry` contract that accepts messages from an enshrined L1 contract, and initializes
L2 user set. Attributes for each DOPE token are stored here.
- `DopeWarsV1` contract.
    - May call the UserRegistry, selecting attributes for use in the game. (Player customisation
    pre-game)
    - Stores game state
    - Calls AMM contract for trades.
- `MarketMaker` contract.
    - Contains the mechanics for trades. Accepts values for market inventory and user action,
    executes the trade, returns the values to the game contract.

### Onboarding

Two potential mechanisms for getting DOPE NFT characteristics into L2:

1. `claimPlayer` L2 Cairo contract that allows user to start on L2 by submitting a merkle
proof of ownership of an L1 DOPE NFT.
2. `L1toL2GamePortal` L1 Solidity contract that acknowledges the ownership of DOPE and sends a
message that L2 can interpret for onboarding/initialization. May accept non-DOPE holders via
some PAPER mechanic also.

## Game state

- World
    - `gameEnds`, block height of end of game.
    - (`v2`) The potency of effect of a trait on probabalistic events.
        - `traitID` -> `traitPower`.
- Markets
    - Item mappings. Each market has an AMM pool for every item.
        - `itemID` -> `marketID` -> `itemQuantityHeld`
        - `itemID` -> `marketID` -> `itemMoneyHeld`
    - Location mapping. ID is obtained by location.
        - `cityID` -> `suburbID` -> `markedID`
- Players
    - Ten item mappings and one money mapping.
        - `itemID` -> `quantityHeld`
    - (`v2`) Trait mappings, one per 'slot'.
        - `slotID` -> `traitID`

## Game progression

A player taking a turn updates their state, their market's state
and potentially a regional/global state if they trigger some
probabalistic event (supply shock).

## Randomness

Events are triggered probabalistically. Mechanism TBD.

## Player actions

Players take one action, which is a composite of move-trade.

- `moveTo`, a city-suburb tuple `(6,3)` takes them to city `6`, suburb `3`.
- `buyOrSell`, binary option `0` for buy, `1` for sell.
- `itemID`, number in range `[1,9]`. E.g., `3` is a particular item across all markets.
- `itemQuantity`, units to buy/sell. E.g., `12` units of item `x`.

## Contracts

Implementation and design notes:

- Core
    - `UserRegistry.cairo`, design [here](./user_registry.md).
    - `GameEngineV1.cairo`, design [here](./game_engine_v1.md).
    - `MarketMaker.cairo`, design [here](./market_maker.md).
- Onboarding
    - Option 1: Merkle-based L2 claim. Downside: Requires StarkNet to verify Eth1 signature
    (pending StarkNet implementation).
        - `ownershipProof.cairo`, design [here](./ownership_proof.md).
    - Option 2: L1-L2 message bridge. Downside: Requires an L1 transaction.
        - `L1toL2GamePortal.sol`, design [here](./l1_to_l2_game_portal.md).

## How StarkNet works in brief

You install cairo-lang, use the command line tools to compile, test and deploy the contracts.
The StarkNet operator(s) will receive the contract, include it in a block. They will turn the
contract (which is a Cairo program) into a special structure called a trace. The trace is used
to build a proof. The proofs are aggregated and sent to a solidity contract on Ethereum.
The contract checks the proof and walks the StarkNet state forward. It stores enough data to
allow anyone else to exit the L2 if StarkNet collapses.

## Infrastructure

Frontends, chain monitoring of events (pending StarkNet implementation), etc. will
be desired (but not required).
