---
layout: page
title:  "Deployment options"
permalink: /cairo/deployment-options/
toc: false
---

Cairo programs can be deployed in different ways:
- SHARP
    - Custom applications with manual state management
    - This includes all current StarkEx applications
- StarkNet
    - Custom applications with inbuilt state management

The main difference is that StarkNet is an improvement that allows easier application design.
The state of an application is made accessible within a Cairo program. This comes at a cost,
which is a loss some of the features of the Cairo programming language.

## SHARP (Foundations release)

Shared Proving Service (SHARP) allows a Cairo program to be converted into a proof
and stored in the verifier contract. The proofs of different programs are all stored in the same
contract and can be individually interrogated with the `isValid()` function. Contracts
can be deployed to interact with specific programs by using the unique hash of that program.

As such, SHARP bundles together different programs that otherwise have no interaction with each
other. Additionally, it is agnostic as to the sort of application that is deployed.
For example, StarkEx and another program may both have a proof bundled together and submitted
in the same Ethereum transaction.

StarkEx is a contract which is deployed in production on Ethereum mainnet. It is designed
for trading applications and acts as the interface between a user and the proof system.
A user may interact with the StarkEx contract directly to lock funds. Those funds are then
available as a layer two asset. Cairo programs can use those funds to trade assets, and this
activity is then recorded as a proof which is sent, through SHARP, to the on chain verifier.
The StarkEx contract queries the verifier to learn about the trade(s) that occurred.
Simultaneously, any other Cairo program proof may "ride" in the same transaction and
be used in another program (E.g., a competitor trading contract SuperTraderEx).

In summary, SHARP-based programs involve:
- Sharing of the costs of proof submission to Ethereum (Ropsten and Mainnet).
- Fully customisable application design.
- Full access to hints, a facet of Cairo programming that is a complex but powerful
    and expressive feature.
- Full use of the Cairo common library.
- Manual management of the state of an application. (E.g. An application designer must
architect how a Cairo program becomes aware of the current state of the system?).

This last point is where StarkNet has an advantage. StarkNet allows a Cairo program to
read values stored by previous proof submissions. In SHARP-based applications, this information
must be encoded in the `input.json` file supplied to a Cairo program at the time of proving.

## StarkNet (Planets, Constellations & Universe releases)

StarkNet which is being rolled out in multiple phases, allows a Cairo program to be converted
into a proof and stored in the verifier contract. This is the same contract used to store
proofs created using SHARP. Similar to SHARP-based programs, a call to the verifier contract
allows the proof of an individual application to be interrogated based on its hash.

StarkNet is a network, which means that it has the ability to perform actions and
complex coordination tasks. A StarkNet operator may take a Cairo program and perform some
task before the program is converted into a proof. This is powerful because it allows the
operator perform increasingly complex tasks to enrich the layer 2 system. Features for
the different stages of the roadmap are as follows:
- **Planets**. Read the Cairo program and insert state values requested, such as the current
balance of an address. For example, the state implied after previous state updates
are applied through previously submitted proofs. Seamless access to storage values is
the main benefit the StarkNet Planets Alpha release offers.
- **Constellations**. Read the state of another Cairo program (L2->L2 one-way interaction)
or interact with another Cairo program (L2<->L2 two-way interaction). This allows actions
similar to the rich contract-to-contract interactions present on mainnet Ethereum.
- **Universe**. Deployment of the operation of StarkNet system as a decentralised public good,
allowing a robust and diverse set of operators to maintain the tasks of the system. In this
stage, interaction with StarkNet will be akin to present-day Ethereum mainnet, where there is
a flexible competitive protocol that is decentralised and which provides censorship resistant
transactions to end-users.

In summary, in the latest Planets Alpha release, StarkNet-based programs involve:
- Sharing of the costs of proof submission to Ethereum.
- Automatic state access management allowing for an "it just works" deployment,
compared with SHARP based applications requiring more complex systems.
- No hints (this makes the StarkNet operator role safer).
- Limited subset of the Cairo common library (due to lack of hints).
