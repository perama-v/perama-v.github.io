---
layout: page
title:  "Technical Summary"
permalink: /cairo/technicals/
toc: false
---

An exploration of the high level concepts important within Cairo and the broader
ecosystem in which Cairo operates. The topics start with foundation concepts
before moving to more Cairo-specific topics.

Ethereum
--------

Ethereum is a blockchain with a virtual machine that can execute arbitrary code.
It has 14 second blocks that contain transactions which execute opcodes to modify the
storage and state of nodes within the network. Each transaction is metered by the opcodes
it uses as a way to quantify the computation and storage requirements it invokes. The more
resources a transation uses, the more it costs.

The metering occurs in a unit called gas, where each block contains about 15 million gas.
A node processing 15 million gas worth of transactions will take a predictable amount of
computational resources. Increasing the amount of gas per block increases the number of
transactions and also the resources required by nodes every block.

Smart Contracts
---------------

Every transaction in Ethereum is a smart contract, which is a sequence of opcode instructions.
Opcodes such as ``PUSH1``, which pushes a 1 byte value onto the stack, are usually abstracted
away with higher level languages such as Yul (intermediate), Solidity (high level, turing complete)
or Vyper (high level, security focused).

Solidity is a statically-typed, object-oriented language. Being Turing-complete, arbitrary programs
can be written and they can interact with other contracts to create complex systems. Contracts
can hold values in memory temporarily during execution or in storage, which persists as state
on every node in the network between contract executions. A contract function is called by sending
a transaction to that contract along with any data the function requires.

Decentralization
----------------

Ethereum full nodes contain the full history of every state transition. Every block since 2015
can be checked to make sure that no transactions violated the rules of the network. Those
\>10 million blocks occupy on the order of hundreds of gigabytes of storage and contain all the
necessary information to independently verify every state transition. Once a state transition has
been checked, any information not required for verifying state transitions is deleted, but may be
recomputed independently later if needed. As new blocks are created, each node checks the
transactions, updates the state and stores the block information.

Nodes can be run with consumer hardware and internet connections with standard bandwidth. For this
reason, more people are able to participate than if specialised hardware and bandwidth was
required. A high node count increases the resilience of the network to shutdown.

Scaling
-------

Increasing the capacity of Ethereum by increasing the gas limit per block increases
node hardware and bandwidth requirements and sacrifices the decentralization property
that makes the network important.

An upgrade termed "sharding" is being actively developed to increase the throughput of the
network. The change involves splitting the chain data into parallel pieces where each node
only verifies subset of the whole. Nodes can demonstrate that they posess data by using
erasure coding and by responding to data sampling queries by the network. This gives a
probabalistic guarantee that the data exists in the network.

Another approach is to use Ethereum to process the state transitions of systems that operate
outside the virtual machine. Each transaction can represent the net outcome of an arbitratry
number of intermediate state transitions. If a set of transactions can be represented by
a transaction with a smaller quantity of gas, more gas is available for other users. This
is a broad description of layer two systems.

Layer Two (L2)
--------------

If Ethereum is a layer one (L1) system, a layer two (L2) system is a network that inherits the
same security guarantees that L1 has. That is, that individual users have the power to control
their assets without requiring the cooperation of a third party.

L2 systems require that L1 has the ability to interpret L2 state transitions. L1 secures L2
because a user can present an L2 state transition to L1, which will then execute a state change
to reflect that update. L2 users have the same guarantees that they have when sending a
normal transaction on L1.

The L1 contracts that accept, process and apply L2 state transitions vary between different
L2 architectures. Their commonality is that a user in posession of their own data can,
if needed, send a transaction to exit the L2 and receive their assets on L1. They do not need
to rely on another system being available or honest.

Rollups
-------

Rollups are a subset of L2 systems that regularly post L2 transactions to L1 as ``CALLDATA``,
which is read-only. L2 transaction data is therefore always available to a full Ethereum node.
Data is stored in a compressed representation that can be accessed by a smart contract if needed.
A merkle tree representing the state of all accounts and balances can be reconstructed from
the data.

Transactions in a rollup are cheaper for two reasons:

-   Less data must be stored on chain. Some data such as the address and
    signature data can be omitted. This makes the size of a regular transaction ~10 bytes
    rather than ~100 bytes.
-   Computation does not need to happen on chain. An L2 transaction that involves a lot of
    computation does not need to pay for that computation on L1 to the same extent.

Every rollup system must have a mechanism to ensure that the computation for each transaction
was correct. This mechanism is enforced by a set of contracts that performs this check.

Two main rollup architectures exist to enforce computation:

-   **Rollups with Fraud Proofs**. Require that anyone who posts L2 data also posts a bond
    which may be slashed if the data does not match the rules of the rollup design.
    The slashing may happen during a days-long window before the L1 data is considered final.
-   **Rollups with Zero Knowledge Proofs**. Create and batch proofs that are posted to L1.
    A smart contract can verify the proof and execute state transitions instantly. Cairo
    uses this architecture.

Rollups have the advantage over state channels, another common L2 construction,
because they do not require as much capital to be locked up in channels.

Field Arithmetic
----------------

Field arithmetic is used in normal blockchain operations, such as for ECDSA-based
signature operations in Ethereum. Field arithmetic is also used in Zero Knowledge Proofs
such as SNARKs, and to a lesser degree, STARKs.

In field arithmetic, numbers are defined as elements within a discrete set. These elements
have properties that allow constructions where:

-   Very large numbers can be used and operations with those numbers in computer programs
    is both fast and precise.
-   Elements within a field can be demonstrated to have desired properties based on the field
    they are a member of.

In Cairo, a program is compiled to a form where individual components can be represented as
elements within particular fields. Constraints can then be applied to those elements to
assert that the state transition throughout a program are correct according to that program.

An integer within a Cairo program is actually a field element, which means that it is an
element of a finite field. A finite field is constructed using modular arithmetic where the
modulo is a prime number. For example, the prime number ``2**64 + 13`` is used in some
parts of Cairo as the basis for representing field elements.

Zero Knowledge Proofs (ZPKs)
----------------------------

Zero Knowledge Proofs allow someone to verify that some computation is correct without
having to do the computations. The technique involves recognising that a computation is
really a mathematical function. To perform computation on some data is the same as
performing a mathematical function ``f(x)`` on that data. A set of checks can be created
that can be given to another person who can very quickly check them and be convinced that
the computation was correct.

There are four main steps:

**1. Encoding**. Demonstrate that a function (computation) was performed on some data,
by constructing an equation of the form:

``a(x) * b(x) = c(x) * d(x)``

Where the functions are derived from the program being proven, and the expression evaluates to
``True`` if the computation was used to derive the outputs presented.

**2. Sampling**. Pick a value for ``x`` and make sure the equation holds, repeat with
some other values. Doing this a few times is much faster than other ways to check the equation.

**3. Encryption**. Hide the functions by applying another function ``E(x)`` to each one. This
stops anyone from learning functions ``a`` to ``d``. The rationale is that:

``a(x) * b(x) = c(x) * d(x)`` is basically the same as ``E(a(x)) * E(b(x)) = E(c(x)) * E(d(x))``

**4. Permute**. Multiply the functions by a secret number, ``k``.

``a(x) * b(x) * k = c(x) * d(x) * k``

Passing someone a set of equations based on the above steps allows them to agree that
the output is truly the result of running the the program with a particular set of inputs.

STARKs
------

STARKs (Scalable Transparent ARguments of Knowledge) are a subset of ZKPs. They
are novel firstly for their absence of a trusted setup and secondly for their
resistance to quantum computers. Both of these features arise from the construction
being created from hashing and data sampling, rather than on elliptic curves and
the private knowledge of some exponent. STARKs are succinct meaning that verification of
a proof is much quicker than the original computation.

STARKs involve taking some computation and representing it as a polynomial function.
Sharing enough points on the line of that function allow anyone to reconstruct the line, similar to
sharing how three points allow anyone to recover a degree-2 polynomial ``(E.g., y=ax^2+bx+c)``.
If one of those three points was changed a very small amount, the line would be mostly the
same. The person could request additional sets of three different points to be sure you
knew the function. With STARKs, the polynomials are used such that if one point is
changed the effects are multiplied and easily detected.

Rather than challenge-response, the challenges are defined by a Merkle tree and you can
send all the sets of points once without the needed for an interactive challenge-response
game.

Where other ZKPs have previously used a global parameter with secret information (a
trusted setup), STARKs use a cryptographic hash function as that global parameter. As
such, there is no risk that secret information is revealed, allowing for a proof to exist
for a false statement. More information can be found in this
[blog post](https://medium.com/starkware/stark-math-the-journey-begins-51bd2b063c71).

AIRs
----

A computation can be reduced to a set of steps, where each step can be checked
for computational integrity. Each step can be represented in a form defined in terms
of polynomials. This polynomial form is an algebraic intermediate representation (AIR)
of that computational integrity statement. Read more about this more in this post about
[arithemtization](https://medium.com/starkware/arithmetization-i-15c046390862).

Expressing computational integrity in terms of polynomials allows for probabalistic
testing to be performed, a key step in generating a STARK.

More specifically, the AIR can be tested easily and quickly to ensure that
it's polynomials are of low degree. This is an important quality to have for it enables
fast proof generation (linear with computational complexity) and verification
(logarithmic with computational complexity). See this
[post](https://medium.com/starkware/arithmetization-ii-403c3b3f4355) for
more details on how and why this is important.

One critical insight is that an AIR is a format for representing computation.
Two computations, each represented as an AIR, can be combined into
a single representation. Combining and arranging AIRs is the basis for
Cairo, as the next section details.

CAIRO
-----

Cairo is a language that stacks together smaller AIRs. Where
a single AIR is the representation of a piece of computation,
multiple AIRs represent arbitrary computation.

The hardware analogy is useful:

- ASIC (AIR)
- CPU (Multiple AIRs)

The name comes from: a CPU built from AIRs (CPU-AIR, Oh nice --> CAIRO).

CAIRO is a non-deterministic, turing complete, functional high level language. It has
a register-based memory model and a compiler. The compiler produces a table of
computational steps called a trace. The trace is used by the prover to construct
AIRs which are combined, subjected to FRI testing (see below), and converted into
a STARK proof.

FRI
---

Cairo organises high level code into AIRs which can be combined for testing.

That testing involves checking that sets of points are valid points on the polynomials in the AIRs.
By using error-checking codes (E.g., Reed-Solomon), oversampling provides assurances about
the polynomials that the points are promised to be on. Applying a restriction to the sampling
allows a recursive method whereby every step further reduces the number of steps required,
similar to how Fast Fourier Transforms are efficiently calculated. The end result is a proof
that a set of points lie on or very close to a line, with a high degree of confidence.

That process is termed FRI (Fast Reed-Solomon Interactive Oracle Proof of Proximity):

- F(ast-fourier-transform style recursive technique)
- R(eed-Solomon-style error checking technique)
- I(nteractive Oracle Proof of Proximity to a low degree polynomial by probabalistic testing)

That is to say that:

1. There is a computation to prove the integrity of.
2. AIRs are a format that allow for a test.
3. The test is required to enable fast proof properties of the STARKs.
4. The test is called "low degree testing".
5. The test is performed with a technique called FRI.

The FRI is then used to construct the STARK proof. Read this
[post](https://medium.com/starkware/arithmetization-ii-403c3b3f4355) for more details.

Prover
------

The prover accepts the Cairo execution trace, including public and private inputs,
and produces a STARK proof. The trace is a table of sequential states for a Cairo program.
For each state transition, a set of constraints is applied to ensure tha the transition is
valid for that element (E.g., each intermediate step in a hash function has been executed
correctly).

The prover:

1.  Creates a statement of integrity for the steps in the trace.
2.  Represents those statments as AIRs (statments in polynomial form).
3.  Combines them into a single AIR.
4.  Performs low degree testing (FRI) to generate a probabalistic proof (that the polynomials
    have low degree).
5.  Generates Merkle trees that commit to that proof.
6.  Saves the commitments as CALLDATA in the verifier smart contract

A more detailed account can be found
[here](https://medium.com/starkware/a-framework-for-efficient-starks-19608ba06fbe).

Note that the prover effectively runs the Cairo program. Being nondeterministic, part of the
role of the prover is to supply inputs to the program. The choice that the prover can make
for those inputs is determined by the Cairo code. Inputs that cause Cairo code assertions
to fail will result in an invalid Cairo program - for which a proof cannot be generated.
Thus, the prover has limited power in this regard and is incentivized to use inputs that
pass checks, allowing them to complete their role as prover.

The prover role is non-trivial in that:

-   For maximum efficiency, it requires the coordination between different parties
    to combine the proofs for two Cairo applications into one.
-   The hardware required to generate a proof is higher RAM and CPU than an average consumer
    computer.

The prover role is therefore performed by:

-   Starkware, now, to support early in-production systems and as an ecosystem support.
-   Starknet users, later, when when infrastructure has been deployed to enable third
    parties to provide the service directly to users under the Starkware Polaris_ license,
    which can be compared to other licenses_ used by Starkware.

Verifier
--------

The overall purpose of a verifier is to assert that ``A(x) = y, within T steps``.
That is, that a program ``A``, with inputs ``x`` produces outputs ``y`` within the time
bound ``T``.

The verifier is a contract which contains the necessary code to verify
a proof. Proof data is sent as a function argument to ``verifyProofAndRegister()``
and the contract processes the proof in a single transaction costing about 1.7 million
gas.

The verifier checks the proof, which has the elements:

-   A set of Merkle roots.
-   Spot-checked branches, picked using the Merkle roots for pseudorandomness
    (Fiat-Shamir Heuristic)
-   A proof that the polynomials representing the Cairo program are of low-degree.

Having checked the proof, the verifier registers the hashes of the programs in a Merkle tree
in storage. Another contract can call ``isValid()`` and learn whether a given hash
was proven or not.

The verifier can be seen on
[testnet](https://ropsten.etherscan.io/address/0xf0EC41069A89595ADf5f27A4a90ff2DF30D83d2E#code)
and
[mainnet](https://etherscan.io/address/0xd4cf925b9d0f4d1ccf82ab97c25130657474ee19#code).

Privacy
-------

The inputs to a given Cairo program are required to generate a proof. The prover
therefore has knowledge of this information. In the current deployment, this prover
is operated by Starkware.

Privacy can be enabled within the protocol by introducing a proof-generation step on the
user-side of Cairo. Such a proof would attest to the value of the private inputs.
This proof would then be passed, along with the usual Cairo program trace to the prover
(either the Starkware proving service, or an independent Starknet prover) who would construct the proof using those components. Privacy is not currently in deployment.
