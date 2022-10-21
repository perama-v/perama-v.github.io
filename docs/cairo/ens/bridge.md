---
layout: page
title:  "ENS"
permalink: /cairo/ens/
toc: false
---

### ENS names on StarkNet - A glimpse at bridging worlds.

Ethereum Name Service (ENS) exists as a registry on L1 that has widespread integration.
L2 networks can be the subject of an ENS resolution, allowing for ENS names to be used
without creating a new conflicting naming system.

The mechanism involves storing name data on the relevant L2 (e.g., a mapping of a name
to an L2 address is stored in a contract on StarkNet). This is made visible on L1 through
a gateway protocol called Durin.

Use case: Existing L1 ENS name owner connects their L2 address.

- Alice owns `alice.eth` on L1 Ethereum, which currently points to their L1 address `0x1234`
through a `resolver_A` contract on L1.
- They want their address to instead point to their StarkNet L2 address `98765`.
- They change a record in the ENS registry so `alice.eth` uses `resolver_B`, which
correctly points to their L2 address using a gateway protocol.

Use case: New L2 user obtains ENS subdomain name.

- Bob wants to make some useful names available in StarkNet. They buy `starknetwallet.eth`
and set the resolver to `resolver_B`.
- They point the name to their StarkNet L2 address `57689`.
- They create a subdomain registry contract on L2 and allow users to register a name in the form
`username.starknetwallet.eth`
- They set the L1 ENS name owner to `0x0000` to prevent the name from being modified and
disrupting the L2 subdomain users.
- The L2 subdomain users may call a function in the L2 subdomain registry that calls L1 and
registers their name there as well (requires custom contract).
- A new user Carol comes to StarkNet and claims `carol.starknet.eth`. Their friend wants to send
them ETH to their L2 address. They call the L1 contract with `carol.starknet.eth`, which
uses a gateway protocol to return Carol's L2 address.

The sections below explore the mechanism by examining the deployment demonstration on
L2 Optimism.

### L1 <-> L2 Bridge background

- Vitalik's FEM
[initial design concept](https://ethereum-magicians.org/t/a-general-purpose-l2-friendly-ens-standard/4591)
- Nick's general purpose bridge
[blog post](https://medium.com/the-ethereum-name-service/a-general-purpose-bridge-for-ethereum-layer-2s-e28810ec1d88).
- Specification [EIP-3668](https://eips.ethereum.org/EIPS/eip-3668). Durin: Secure offchain data retrieval
- ENS's demo code for [Optimism L2 Gateway](https://github.com/ensdomains/l2gateway-demo).

### Components

- New L1 resolver contracts
    - One L1 contract per L2.
    - User with `.eth` points their name to this new resolver.
    - This resolver is L2 specific (E.g., one for Optimism, one for StarkNet)
    - If `alice.eth` is pointed at the Optimism resolver, it will return an address on
    Optimism.
    - Generates data that can be send to the gateway.
    - Accepts and digests responses produced by the gateway.
    - Verifies the integrity of the gateway responses. Is able to determine valid responses
     particular to each L2.
- New L2 registry contracts
    - One L2 contract per L2.
    - Stores a `.eth` name with an associated L2 address.
    - Able to respond to gateway requests with a data that L1 can digest.
    - Stores name data, which is likely cheaper than doing so on L1.
    - Could store many subdomains that never touch L1 (a.wal.eth, b.wal.eth, ..., zzz.wal.eth).
    - Relies on L1 for authority. If `alice.eth` is sold and pointed to a different resolver on L1,
    the L2 record of that name to an L2 address is not going to be used.
- Gateway service
    - One service can handle multiple L2s.
    - Can be called with data produced by the L1 resolver contract.
    - Is operated by some third party as a service or public good.
    - Responds to queries by interpreting the relevant L2 specified, calling the
    L2 registry contract to retrieve an L2 address, responding to the user.
    - The response generated can be sent by the user directly to the L1 resolver contract.


### Moving a name to L2

High level steps to move `alice.eth` from L1 Ethereum to L2 StarkNet:

- Alice changes the resolver that decides the address their ENS name points (resolves) to.
- Alice sends a transaction on L1 that emits a message L1->L2 that registers `alice.eth`
and the L2 address it should point to.
- Alice shares their ENS name with another user, who then seeks truth from the L1 ENS registry.
- The registry points to the new resolver, which then responds with a gateway URL and a
payload to send.
- User sends payload to gateway.
- Gateway reads which L2 is specified and which address and function to call on L2.
- Gateway calls the function on L2 contract to discover the L2 address Alice has stored.
- Gateway receives the result from the L2 call.
- Gateway responds to the user.
- User accepts the data and passes it to the L1 resolver contract.
- The contract verifies the response is valid
- The L2 address is provided as the result.

More granular steps for the above process, following this
[deployment sequence](https://github.com/ensdomains/l2gateway-demo/blob/appgateway/contracts/scripts/app_deploy.js).

**L1 actions**

- Own an existing ENS name in the ENS registry.
- Deploy a resolver contract on L1 that contains the URL of the gateway.
- The owner of an ENS name calls `setResolver()` function of the L1 ENS registry contract.
- L1 user sets the resolver in the ENS registry to a new L2-specific resolver contract like
[this one](https://github.com/ensdomains/l2gateway-demo/blob/appgateway/contracts/contracts/l1/OptimismResolverStub.sol),
- They generate an L1 transaction the creates an L1-to-L2 message which stores the name
`alice.eth` and the L2 address they control as a mapping inside an L2 registry contract
like
[this one](https://github.com/ensdomains/l2gateway-demo/blob/appgateway/contracts/contracts/l2/OptimismResolver.sol).

**L2 actions**

Deploy a contract to StarkNet that can store mappings from names to addresses.
Names are stored as a `node`, which is a hash-based representation of the name.

This is an example contract from
[ENS L2 MVP Demo](https://github.com/ensdomains/l2gateway-demo/tree/appgateway/contracts/contracts/l2)
which outlines a contract that would be deployed to Optimism (L2).
```
pragma solidity ^0.7.6;

import "@openzeppelin/contracts/access/Ownable.sol";

contract OptimismResolver is Ownable {
    mapping(bytes32=>address) addresses;

    event AddrChanged(bytes32 indexed node, address a);

    function setAddr(bytes32 node, address addr) public onlyOwner {
        addresses[node] = addr;
        emit AddrChanged(node, addr);
    }

    function addr(bytes32 node) public view returns(address) {
        return addresses[node];
    }
}
```
The above contract returns an `address` on L2 for a given name (`node`) when the
`addr()` function is called. It can also
update the L2 address stored when the `setAddr()` is called. The equivalent contract
would be written in Cairo and deployed to StarkNet, potentially triggered by an L1->L2 message
transaction.

After it is deployed, its address would be stored in the custom resolver contract on L1
used for that particular L2 (E.g., the Optimism address is stored in the OptimismResolver
contract).

**L1 Actions**

- Anyone curious about `alice.eth` calls the
[ens registry](https://github.com/ensdomains/ens/blob/master/contracts/ENSRegistry.sol)
contract `resolver()` function with the node (
[namehash](https://docs.ethers.io/v5/api/utils/hashing/#utils-namehash) of `alice.eth`). This
returns the address of the resolver contract that Alice has chosen (E.g., the custom StarkNet ENS resolver)
- The `addr()` function of resolver contract is called, passing the `node` as an argument.
This returns data (representing an application-specific function name prefix and the
name `alice.eth`) and a `url` (or multiple URLs.).
- The user sends that data to the url which is a third party gateway provider.

**Gateway Actions**

- Await calls from users that contain data in an appropriate format, interpretable as a
particular L2, an L2 contract address where names are stored and data representing the name
query.
- Accesses the L2 chain, calls the contract to retrieve the data, which includes
a mechanism to prove that the data is valid. This might take the form of a Merkle proof.
- Returns the data and proof to the user.

The Optimism state roof proof has the following components:

```
  struct L2StateProof {
    bytes32 stateRoot;
    Lib_OVMCodec.ChainBatchHeader stateRootBatchHeader;
    Lib_OVMCodec.ChainInclusionProof stateRootProof;
    bytes stateTrieWitness;
    bytes storageTrieWitness;
  }
```

**L1 Actions**

- User calls the `addrWithProof()` function in the
[resolver contract](https://github.com/ensdomains/l2gateway-demo/blob/653f41b371fd82e9ff9e59dad235516ea8dc2368/contracts/contracts/l1/OptimismResolverStub.sol#L32),
passing as arguments the ens node and the proof data from the gateway response.
- The resolver contract internally verifies the proof with `verifyStateRootProof()`,
by calling the `verifyStateCommitment()` function in the L1 Optimism
[StateCommitmentChain](https://github.com/ethereum-optimism/contracts/blob/master/contracts/optimistic-ethereum/OVM/chain/OVM_StateCommitmentChain.sol)
contract. This involves verifying a the branch of a Merkle tree for a given leaf hash using a
[library](https://github.com/ethereum-optimism/contracts/blob/master/contracts/optimistic-ethereum/libraries/utils/Lib_MerkleTree.sol).
- The resolver then gets the L2 address from the proof data with the `getStorageValue()`
[function](https://github.com/ensdomains/l2gateway-demo/blob/653f41b371fd82e9ff9e59dad235516ea8dc2368/contracts/contracts/l1/OptimismResolverStub.sol#L44).

**L2 Actions**

The user then may send value to the L2 address that Alice owns, having only known the
string `alice.eth`.

## Deploying subdomains directly to L2

The owner of an ENS name (E.g., Bob owns `starknetwallet.eth`) can point the name to
the L1 StarkNet resolver contract. They can then send a message from L1->L2 signaling
that subdomain registration is permitted on L2.

The L2 registry contract that stores all the names (E.g., `alice.eth`) also stores the rules that
the owner has set about subdomain registration (Eg., who can register, cost, etc.). An
L2 user can follow these rules to set up an L2 ENS name (`carol.starknetwallet.eth`)
that can be retrieved from L1 like in the previous example.
This avoids L1 interaction by the user. Note that if Bob sets the
owner of the `starknetwallet.eth` to 0x0000 on L1, then the L2 subdomains cannot be revoked
by Bob changing the L1 resolver to another contract (E.g., the Optimism L2 ENS resolver)

## Message mechanisms

The protocol requires that the L1 contract verifies the proof that the L2 state includes
the ENS name-address mapping. The Durin gateway protocol involves read-only contract calls.
This means that a proof may be verified by an ethereum L1 read-only transaction, which
is free.

A user who makes a query to an Ethereum L1 node to find the address (L1 or L2) associated
with an ENS name does not pay for an L1 transaction. They do send a query to a gateway,
which may charge a fee.

As new users interact with the L2 ENS contract by sending StarkNet transactions, the
StarkNet state is updated. These updates are included in L1 transactions, making proof data
available on L1 for the resolver during the `addrWithProof()` call that follows the
gateway query.

One mechanism is for the StarkNet contract that acts as the L2 ENS registry
to have a variable that stores some state reference. When the Durin gateway operator calls
the contract asking for the address associated with the ENS name, the state reference is
provided as part of the response. The contract could provide enough data to make a
Merkle proof that the ENS name has been included in a state that has been proven
and secured on L1. This would mean some delay (E.g., 1 hour) before an ENS name
reflects the most recent L2 address data. However, it is a good tradeoff, because the
proof data cost is amortised (and paid in part by the L2 user setting their name-address
mapping with a StarkNet transaction) and the proving cost on L1 is free (read-only).