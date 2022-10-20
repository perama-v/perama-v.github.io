---
layout: page
title:  "Descriptive Summary"
permalink: /cairo/description/
toc: false
---

An exploration of the high level concepts important within Cairo and the broader
ecosystem in which Cairo operates. The topics start with foundation concepts
before moving to more Cairo-specific topics.

Ethereum
--------

Ethereum is a platform shared across many computers around the world. It allows
different people to come to agreements despite not knowing or trusting each other.
These agreements allow you to send and program money. The core reason why Ethereum is
powerful is that anyone can design a program and publish it for anyone else to use.
These programs can be crafted to perform a myriad of tasks, and can be thought of as
digital robots that coordinate value between people all around the world.

Each Ethereum transactions costs money to incentivise people to keep to the network running
on their own computer. A very large Ethereum program or transaction causes those computers
to work harder, and therefore costs more money that a small transaction. Ultimately, there is a
limit to how hard you can make each computer work, and because of that, the network accepts
fewer than ~1000 transactions per minute.

Smart Contracts
---------------

Every transaction on Ethereum is a program which instructs the network to perform some task.
One such task might be "Send 5 coins to Alice if Bob sends me 20 coins.". These programs are
written in a programming language such as Solidity, which is the most popular. Every computer in
the network runs the program and confirms that Alice received the coins according to the program.

Programs can be complex, and can involve storing information for later use. A program might
store the exchange rate between two currencies, and later, a user might use that program to
swap one currency for the other. Another program might detect that swap and take some
other action. Anyone can write and deploy a program.

Decentralization
----------------

Blockchains are powerful because anyone can join with their own computer and check
that everything is operating as it should. They don't have to believe anyone else that
their money is stored in a particular program because their computer has run the program
and confirmed that to be true.

If the blockchain was too large for their computer to process, they would not be able to do that.
For this reason, it is important to make blockchains small and easy to operate on a normal
computer. The more people that participate, the more copies of the blockchain exist and the
higher the chance that the network will continue to exist. If someone tries to shut
down Ethereum, they will have to work harder if there are thousands of computers
to find and control than if there were tens or hundreds.

Scaling
-------

In order to enable many humans to use Ethereum, the capacity of the blockchain must increase
without sacrificing the decentralization property that makes it so important.

One avenue to achieve that is by distributing the blockchain into many pieces,
similar to how one can use file sharing software (E.g., torrent software) without
storing all the files. This approach is being engineered for Ethereum as a future
upgrade called "sharding".

Another avenue is to look the burden of each transaction on Ethereum and consider
different ways to store that information. For example if Alice and Bob send each other
coins all day, the important information to store might be the net outcome, rather than
each and every transaction. The final outcome, Alice sends Bob 10 coins, has a smaller
computational footprint than their full list of 100 transactions. That unused space can
then be used by other participants.

Layer Two (L2)
--------------

A blockchain, such as Ethereum, is commonly referred to as a layer one (L1) system.
A layer two (L2) system is a network that accepts and processes transactions, but
ultimately uses L1 as the final source of truth.

Alice and Bob may both agree to use an L2 to save money. On L1 they send their coins to a
contract designed to handle disputes, this enables them to participate in L2 using those coins.
They then use L2 to send coins back and forth between each other. The L2 tracks all the
transactions they make. If they disagree about who owns what coins at any point, they can
send a message to the contract on L1 to handle their dispute. The dispute contract updates
their L1 balances and neither party has to worry that their coins are going to be locked away
by some system or stranger.

The contract on Ethereum is a program that acts as a courthouse in the event
of a dispute between Alice and Bob about who owns a coin.

Rollups
-------

Rollups are a layer two (L2) technology that involve sending transactions in a system where
they are aggregated and only a very small part of each transaction is sent to Ethereum (L1).

Transactions in a rollup are cheaper for two reasons:
-   Only a fraction each transaction needs to be stored on L1 Ethereum.
-   None of the computation for a transaction needs to be performed on L1 Ethereum.

Rollups are powerful because everything is on Ethereum (L1) and there is no risk of a transaction
being lost or withheld by someone.

The computation for each transaction must still be enforced. There are two approaches for this:

-   Require a bond. If someone posts transaction data that does not follow the agreed
    rules of computation their bond is taken away. This requires a time window for people
    to be able to check data and make such claims. These are called **Rollups with Fraud Proofs**.
-   Use mathematics to prove that the computation was correct. No need to check again
    inside the expensive L1 Ethereum contract. If the proof is posted, the contract knows
    everything has gone according to the rules agreed on by all the L2 participants.
    These are called **Rollups with Zero Knowledge Proofs**. Cairo uses this architecture.

Rollups allow for Alice and Bob to send coins to each other at a low cost. Rollups also allow them
to do other activities with their coins. It is a flexible system that doesn't require them
to set coins aside, reserved only for "paying and receiving payments from people".

Field Arithmetic
----------------

Field arithmetic is a topic within mathematics that is important for both normal blockchain
transactions (public key cryptography) and for the more advanced components that Cairo uses
(zero knowledge proofs).

The core principle with field arithmetic is that there are calculations that are very easy
for a user to do (sign a transaction) but nearly impossible for an attacker (find the secret
key you used to sign a transaction). These calculations are hard for the attacker, in part,
because the numbers involved are very large.

Computers normally process numbers with a "near enough is good enough" approach. Miniscule
errors after a decimal point are usually unimportant. In cryptography this imprecision is
unacceptable. Expanding the key analogy: A key must fit the lock perfectly for a blockchain
to accept it. This problem with precision is solved by using field arithmetic.

Field arithmetic uses a system similar to calculating the time of day in 100 hours time. The time
will always be in the range 1 o'clock to 12 o'clock (E.g., it will never be 85 o'clock).
Field arithmetic ensures that computation with large numbers is always in a certain range.
From this system, special tricks enable computers to rapidly arrive at the answer to calculations.

Field arithmetic is built upon mathematics with procedures involving equations and large prime
numbers. They are important when designing the Cairo system, but are not required for
a developer who wishes to use Cairo to create an application.

Zero Knowledge Proofs (ZPKs)
----------------------------

Zero knowledge proofs are a decades-old invention which has special
privacy-preserving qualities. For many years the work was academic, but in the last ten
years and with an ever-increasing pace of innovation, they have become practical to use.

A zero knowledge proof enables you to say "I know some secret".

A person who doesn't believe you can challenge you and ask you to prove that. Without
sharing the facts, you can create a game that you play with that person many (E.g., 100) times.
At the end of those games, they believe you and agree that you know the secret you promised.
An example of such a game is, A challenger takes the pieces of a puzzle you have made and
either rearranges it, or leaves it. If you can tell when it was rearranged, the challenger
knows that you have the information about the puzzle that you promised.

In zero knowledge proofs, a program performs this game and generates a receipt that
mathematically prooves that the claim "I know some secret" is valid.

STARKs
------

STARKs are a form of ZKP with some nice properties that make them useful for scaling
and privacy. With rollups, transactions can be reduced to tiny summaries that save space.
STARKs are a way to take a program and summarize its operation such that a tiny summary
can be put into the rollup. The power of a STARK is that the summary is always tiny,
even if the program is very large.

Another feature of STARKs is that they eliminate a special type of risk inherent to other
Zero Knowledge Proofs. Proofs are made using a computer program that has been created for
that purpose. Some proof systems require that a special secret be created by many different
people. There is a very small but non-zero chance that all the secret-creators get together
at some point and agree to join forces to steal money.

With STARKs, this requirement is lifted and no special secret ceremony is required to
build the program. In this way it is not reliant on participants keeping a promise to never
share their secret. The program is 'transparent'.

AIRs
----

An AIR is a format that a Cairo program can be transformed into. The main purpose of
a Cairo program is to translate readable code into powerful STARK proofs. The AIR is
an intermediate step in that process.

An AIR can be thought of as deconstructing a program and reconstructing it as a spreadsheet.
Every row and column in the spreadsheet accounts for different parts of the program.
As the program progresses, new lines are added to the spreadsheet.

The spreadsheet allows line-by-line checking of the program. If all the checks pass,
then the program can progress to the next stage of the proof generation.

AIRs are novel in that they allow complex Cairo code to be broken down into small, easy
to reason about regions in the spreadsheet. In that way, a Cairo program can be written
to solve any arbitrary problem, without needing to rewrite the checks that need to be passed.
Each program can instead be loaded into the spreadsheet, reusing complicated hand-crafted
formulae. This saves a lot of time for application developers.

CAIRO
-----

Cairo a language that reads like a familiar computer programming language but
functions like a zero knowledge proof. It allows a developer to write a program
by focusing on the logic of an application, rather than the on the mechanics
of how the application will be converted into a proof.

Cairo programs can be written to solve complex problems, just like most other
common programming languages. However, where those programs finish and produce
an output, Cairo programs produce an output and a proof.

A third party does not need to run the program if they have all of:
1.  Cairo program code.
2.  The proof.
3.  The output.

They can read the Cairo code, easily reason about the program logic,
check the proof is valid and then use the output to make some change. As
proofs are small enough to be saved on Ethereum, that change is usually
an interaction with a smart contract. Cairo programs are ready for
production-level applications.

The zero knowledge proof developer experience can be thought of as two eras:

-   Pre-Cairo: Write custom proofs, a task as complex as building a new computer
    for every computer program.
-   Post-Cairo: Write Cairo programs that can be turned into proofs, a task as simple
    as reusing a computer that was already built and which can handle any program.

FRI
---

FRI is a technique that can be used to show that the building blocks for a
a system are simple, rather than complex. Proofs built from simple building
blocks are useful and practical, whereas proofs built from large building blocks are
impractical and must be avoided in the construction of a STARK proof system.

In this analogy, the building blocks are the AIRs that Cairo programs produce. The
term "simple" refers to the power of elements in the building block, where a number raised
to the power 2 (``x^2``) is more simple than a number raised to the power of 20 (``x^20``).

One technique to prove that building blocks are small is:

1.  A person picks a value and asks you to use the block to find other values.
2.  They check the values, to make sure there is no suggestion that the block is too complex.
3.  Steps 1 and 2 are repeated, until the person is happy that the chance of the building
    blocks being simple is almost 100%.

FRI removes that back-and-forth, and the process becomes:

1.  A random number is used to pick a set of values in advance.
2.  The building blocks are used to find the associated values.
3.  The starting values, and derived values are then given to another person
4.  The person goes through all the numbers and is happy that the chance of the building
    blocks being simple is almost 100%.

The FRI allows for a one-step check that a set of ingredients that came from a Cairo program
have been prepared in such a way they are suitable to be baked into a STARK proof.

Prover
------

A prover is an entity that takes Cairo code and an output and produces
the proof for that specific situation. That is, a user who interacts with
a Cairo program will generate a file that is a recipe for a proof. Anyone
with the right proving software can use the ingredients to calculate the
proof.

The prover is a role that can be performed by different parties. Having a
prover available as a service allows someone to use Cairo in the most simple mode.
The prover accepts the ingredients, calculates the proof and then stores that
proof on Ethereum. That guarantees to the world that the program was structurally
valid and that the output was correctly produced.

The prover role is special for three reasons:

- Proofs are intensive to run. A mobile phone is inadequate for such a task
  and a dedicated computer is a better solution.
- Proofs involve coordination. Multiple proofs can be combined for efficiency,
  but organising how these proofs come to be joined requires some effort.
- Proofs need to go on Ethereum. A proof is sent as a transaction that must
  be created, paid for, and monitored to ensure success.

Provers support the Cairo ecosystem for different reasons:

- Starkware provides a proving service to nurture the system in the early stage.
- Starknet is a system that will allow anyone to provide a proving service for
  some compensation or incentive.

Verifier
--------

The purpose of the verifier is to say that a Cairo program, when fed certain inputs,
produced certain outputs, within a reasonable amount of time.

The verifier is a robotic task that involves checking that some proof is valid.
The verification is simple enough to be performed by a smart contract without
too much expense. Such a contract exists on Ethereum permanently and contains
the information and logic required to digest a proof and return a single result:
Valid or Not Valid.

The contract is triggered by an Ethereum transaction and may interact with other
contracts as part of a larger application.

The verifier contract has the power to say that it has checked that:

1. The Cairo program is properly constructed.
2. The program was run with the inputs provided.
3. The outputs from the program are real and accurate.

The verifier can do this task as soon as the proof has arrived on chain, and at
any point thereafter.

Privacy
-------

A Cairo proof is a small receipt that convinces a third party a program has
been run. The receipt may have that power even if it does not reveal the
inputs to that program. In this way, a user can put secrets into a Cairo
program and be sure that when the receipt is used in public, that the receipt
does not expose those secrets.

Privacy use cases with Cairo are largely of the form:

- A user has private information.
- The information is useful for some purpose.
- The purpose can be represented by a smart contract interaction.

For example, a user may:

- Prove ownership of an asset to trade without public exposure.
- Prove membership of a group to cast a private vote.
- Prove they have complied with some regulation to participate in a fund raise.

This feature is not yet deployed, but can theoretically be added to Cairo in the future
as a second feature beside scaling.
