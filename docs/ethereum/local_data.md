---
layout: page
title:  "Local data"
permalink: /cairo/ethereum/local-data
toc: true
---

Here is a peek at some cool tech built by some great, hard-working people.

Install [Erigon (GNU GPL v3.0)](https://github.com/ledgerwatch/erigon) and run a default node sync
to get a 1.3TB archive node. That includes all manner of information that powers
much of what happens on Ethereum from the user-side. Set up a service where the node
responds to JSON-RPC queries locally.

Install [TrueBlocks (GNU GPL v3.0)](https://github.com/TrueBlocks/trueblocks-core) and initialise
the setup. That pulls bloom filters from IPFS, which create an efficient way to locate
data within an archive node. Among many other lightning-fast capabilities, you can pull
out contract bytecode.

Install [Panoramix (MIT)](https://github.com/palkeo/panoramix), a local bytecode decompiler.

Now suppose a contract is deployed at an address.

`0x1a2a1c938ce3ec39b6d47113c7955baa9dd454f2`

Let us inspect, using only local data, what lies therein. TrueBlocks `chifra state` comes
to the rescue:

```
chifra state 0x1a2a1c938ce3ec39b6d47113c7955baa9dd454f2 --parts all --fmt json --verbose 2
```
Returns *instantly*:
```
chifra state 0x1a2a1c938ce3ec39b6d47113c7955baa9dd454f2 --parts all --fmt json --verbose 2
{ "data": [
  {81398
    "blockNumber": 12781398,
    "balance": 19380817203324916966,
    "nonce": 1,
    "code": "0x6080604052600436106100dd5760003560e01c80637b1039991161007f5780639a202d47116100595780639a202d47146102b2578063b02c43d0146102c7578063f851a44014610331578063fd840de214610346576100dd565b80637b103999146102555780638456cb591461026a5780638f2839701461027f576100dd565b80634555d5c9116100bb5780634555d5c9146101925780635c60da1b146101a75780635c975abb146101d85780635cc0707614610201576100dd565b80631a5da6c8146101215780632dfdf0b5146101565780633f4ba83a1461017d575b60006100e7610379565b90506001600160a01b0381166100fc57600080fd5b60405136600082376000803683855af43d806000843e81801561011d578184f35b8184fd5b34801561012d57600080fd5b506101546004803603602081101561014457600080fd5b50356001600160a01b0316610388565b005b34801561016257600080fd5b5061016b6103c1565b60408051918252519081900360200190f35b34801561018957600080fd5b506101546103c7565b34801561019e57600080fd5b5061016b61042c565b3480156101b357600080fd5b506101bc610379565b604080516001600160a01b039092168252519081900360200190f35b3480156101e457600080fd5b506101ed610431565b604080519115158252519081900360200190f35b34801561020d57600080fd5b5061022b6004803603602081101561022457600080fd5b5035610441565b604080516001600160a01b0394851681529290931660208301528183015290519081900360600190f35b34801561026157600080fd5b506101bc61046f565b34801561027657600080fd5b5061015461047e565b34801561028b57600080fd5b50610154600480360360208110156102a257600080fd5b50356001600160a01b03166104ea565b3480156102be57600080fd5b5061015461056f565b3480156102d357600080fd5b506102f1600480360360208110156102ea57600080fd5b50356105ce565b604080516001600160a01b0396871681529486166020860152929094168383015263ffffffff166060830152608082019290925290519081900360a00190f35b34801561033d57600080fd5b506101bc610623565b34801561035257600080fd5b506101546004803603602081101561036957600080fd5b50356001600160a01b0316610632565b6001546001600160a01b031690565b6000546001600160a01b0316331461039f57600080fd5b600280546001600160a01b0319166001600160a01b0392909216919091179055565b60035481565b6000546001600160a01b031633146103de57600080fd5b600154600160a01b900460ff166103f457600080fd5b6001805460ff60a01b191690556040517fa45f47fdea8a1efdd9029a5691c7f759c32b7c698632b563573e155625d1693390600090a1565b600290565b600154600160a01b900460ff1681565b6005602052600090815260409020805460018201546002909201546001600160a01b03918216929091169083565b6002546001600160a01b031681565b6000546001600160a01b0316331461049557600080fd5b600154600160a01b900460ff16156104ac57600080fd5b6001805460ff60a01b1916600160a01b1790556040517f9e87fac88ff661f02d44f95383c817fece4bce600a3dab7a54406878b965e75290600090a1565b6000546001600160a01b0316331461050157600080fd5b6001600160a01b03811661051457600080fd5b600080546040516001600160a01b03808516939216917f7e644d79422f17c01e4894b5f4f588d331ebfa28653d42ae832dc59e38c9798f91a3600080546001600160a01b0319166001600160a01b0392909216919091179055565b6000546001600160a01b0316331461058657600080fd5b600080546040516001600160a01b03909116917fa3b62bc36326052d97ea62d63c3d60308ed4c3ea8ac079dd8499f1e9c4f80c0f91a2600080546001600160a01b0319169055565b600481815481106105db57fe5b600091825260209091206004909102018054600182015460028301546003909301546001600160a01b0392831694509082169291821691600160a01b900463ffffffff169085565b6000546001600160a01b031681565b6000546001600160a01b0316331461064957600080fd5b6001600160a01b03811661065c57600080fd5b600180546001600160a01b0319166001600160a01b03838116918217928390556040519216917fd32d24edea94f55e932d9a008afc425a8561462d1b1f57bc6e508e9a6b9509e190600090a35056fea265627a7a723158202489cc87c94756bca4cb382b1aa499a969d11bc469ee7a3912746a14c5c7862a64736f6c63430005110032",
    "storage": "0x00000000000000000000000023d4817717fc407ee8266dc45f4f8a1ccc5338fa",
    "address": "0x1a2a1c938ce3ec39b6d47113c7955baa9dd454f2",
    "deployed": 11724674,
    "accttype": "Contract",
    "ether": 19.380817203324916966
  }] }
```

Delicious impassable runtime bytecode. Now for Panoramix:

```
panoramix 0x6080604052600436106100dd5760003560e01c80637b1039991161007f5780639a202d47116100595780639a202d47146102b2578063b02c43d0146102c7578063f851a44014610331578063fd840de214610346576100dd565b80637b103999146102555780638456cb591461026a5780638f2839701461027f576100dd565b80634555d5c9116100bb5780634555d5c9146101925780635c60da1b146101a75780635c975abb146101d85780635cc0707614610201576100dd565b80631a5da6c8146101215780632dfdf0b5146101565780633f4ba83a1461017d575b60006100e7610379565b90506001600160a01b0381166100fc57600080fd5b60405136600082376000803683855af43d806000843e81801561011d578184f35b8184fd5b34801561012d57600080fd5b506101546004803603602081101561014457600080fd5b50356001600160a01b0316610388565b005b34801561016257600080fd5b5061016b6103c1565b60408051918252519081900360200190f35b34801561018957600080fd5b506101546103c7565b34801561019e57600080fd5b5061016b61042c565b3480156101b357600080fd5b506101bc610379565b604080516001600160a01b039092168252519081900360200190f35b3480156101e457600080fd5b506101ed610431565b604080519115158252519081900360200190f35b34801561020d57600080fd5b5061022b6004803603602081101561022457600080fd5b5035610441565b604080516001600160a01b0394851681529290931660208301528183015290519081900360600190f35b34801561026157600080fd5b506101bc61046f565b34801561027657600080fd5b5061015461047e565b34801561028b57600080fd5b50610154600480360360208110156102a257600080fd5b50356001600160a01b03166104ea565b3480156102be57600080fd5b5061015461056f565b3480156102d357600080fd5b506102f1600480360360208110156102ea57600080fd5b50356105ce565b604080516001600160a01b0396871681529486166020860152929094168383015263ffffffff166060830152608082019290925290519081900360a00190f35b34801561033d57600080fd5b506101bc610623565b34801561035257600080fd5b506101546004803603602081101561036957600080fd5b50356001600160a01b0316610632565b6001546001600160a01b031690565b6000546001600160a01b0316331461039f57600080fd5b600280546001600160a01b0319166001600160a01b0392909216919091179055565b60035481565b6000546001600160a01b031633146103de57600080fd5b600154600160a01b900460ff166103f457600080fd5b6001805460ff60a01b191690556040517fa45f47fdea8a1efdd9029a5691c7f759c32b7c698632b563573e155625d1693390600090a1565b600290565b600154600160a01b900460ff1681565b6005602052600090815260409020805460018201546002909201546001600160a01b03918216929091169083565b6002546001600160a01b031681565b6000546001600160a01b0316331461049557600080fd5b600154600160a01b900460ff16156104ac57600080fd5b6001805460ff60a01b1916600160a01b1790556040517f9e87fac88ff661f02d44f95383c817fece4bce600a3dab7a54406878b965e75290600090a1565b6000546001600160a01b0316331461050157600080fd5b6001600160a01b03811661051457600080fd5b600080546040516001600160a01b03808516939216917f7e644d79422f17c01e4894b5f4f588d331ebfa28653d42ae832dc59e38c9798f91a3600080546001600160a01b0319166001600160a01b0392909216919091179055565b6000546001600160a01b0316331461058657600080fd5b600080546040516001600160a01b03909116917fa3b62bc36326052d97ea62d63c3d60308ed4c3ea8ac079dd8499f1e9c4f80c0f91a2600080546001600160a01b0319169055565b600481815481106105db57fe5b600091825260209091206004909102018054600182015460028301546003909301546001600160a01b0392831694509082169291821691600160a01b900463ffffffff169085565b6000546001600160a01b031681565b6000546001600160a01b0316331461064957600080fd5b6001600160a01b03811661065c57600080fd5b600180546001600160a01b0319166001600160a01b03838116918217928390556040519216917fd32d24edea94f55e932d9a008afc425a8561462d1b1f57bc6e508e9a6b9509e190600090a35056fea265627a7a723158202489cc87c94756bca4cb382b1aa499a969d11bc469ee7a3912746a14c5c7862a64736f6c63430005110032
```

Returns:

```
panoramix.decompiler Running light execution to find functions:
panoramix.utils.signatures Cache for PABI not found, generating...
panoramix.utils.signatures Cache for PABI generated.
panoramix.decompiler Parsing updateRegistry(address _registry)...
panoramix.decompiler Parsing depositCount()...
panoramix.decompiler Parsing unpause()...
panoramix.decompiler Parsing proxyType()...
panoramix.decompiler Parsing implementation()...
panoramix.decompiler Parsing paused()...
panoramix.decompiler Parsing withdrawals(uint256 _param1)...
panoramix.decompiler Parsing registry()...
panoramix.decompiler Parsing pause()...
panoramix.decompiler Parsing changeAdmin(address _admin)...
panoramix.decompiler Parsing removeAdmin()...
panoramix.decompiler Parsing deposits(uint256 _param1)...
panoramix.decompiler Parsing admin()...
panoramix.decompiler Parsing unknownfd840de2(?)...
panoramix.decompiler Parsing _fallback()...
panoramix.decompiler Functions decompilation finished, now doing post-processing.
# Palkeoramix decompiler.

const proxyType = 2

def storage:
  adminAddress is addr at storage 0
  paused is uint8 at storage 1 offset 160
  implementationAddress is addr at storage 1
  registryAddress is addr at storage 2
  depositCount is uint256 at storage 3
  deposits is array of struct at storage 4
  withdrawals is mapping of struct at storage 5

def depositCount(): # not payable
  return depositCount

def implementation(): # not payable
  return implementationAddress

def paused(): # not payable
  return bool(paused)

def withdrawals(uint256 _param1): # not payable
  require calldata.size - 4 >= 32
  return withdrawals[_param1].field_0, withdrawals[_param1].field_256, withdrawals[_param1].field_512

def registry(): # not payable
  return registryAddress

def deposits(uint256 _param1): # not payable
  require calldata.size - 4 >= 32
  require _param1 < deposits.length
  return deposits[_param1].field_0,
         deposits[_param1].field_256,
         deposits[_param1].field_512,
         deposits[_param1].field_512,
         deposits[_param1].field_768

def admin(): # not payable
  return adminAddress

#
#  Regular functions
#

def unpause(): # not payable
  require caller == adminAddress
  require paused
  paused = 0
  log Unpaused()

def pause(): # not payable
  require caller == adminAddress
  require not paused
  paused = 1
  log Paused()

def removeAdmin(): # not payable
  require caller == adminAddress
  log AdminRemoved(address address=adminAddress)
  adminAddress = 0

def updateRegistry(address _registry): # not payable
  require calldata.size - 4 >= 32
  require caller == adminAddress
  registryAddress = _registry

def unknownfd840de2(addr _param1): # not payable
  require calldata.size - 4 >= 32
  require caller == adminAddress
  require _param1
  implementationAddress = _param1
  log 0xd32d24ed: _param1, _param1

def changeAdmin(address _admin): # not payable
  require calldata.size - 4 >= 32
  require caller == adminAddress
  require _admin_
  log AdminChanged(
        address from=adminAddress,
        address to=_admin_)
  adminAddress = _admin_

def _fallback() payable: # default function
  require implementationAddress
  delegate implementationAddress with:
     funct call.data[0 len 4]
       gas gas_remaining wei
      args call.data[4 len calldata.size - 4]
  if not delegate.return_code:
      revert with ext_call.return_data[0 len return_data.size]
  return ext_call.return_data[0 len return_data.size]
```

Now *that is interesting*. What of the two mysterious elements:

- `def unknownfd840de2(addr _param1): # not payable`
- `log 0xd32d24ed: _param1, _param1`

How might that blind spot be lifted? Each looks to be a signature, four bytes long.

How about [`Ethereum Signature Database (license-free)`](4byte.directory), with
just under 180k signatures from the first four bytes of the Keccak hash.

Submitting each signature, in turn, to the site.

Yields:

- No results.

A quick search around the internet also ends fruitless.

- The function has a collision with a github commit from may: [...pull/8/commits/fd840de234b8d6...](https://github.com/talibarbosa-hub/projeto-sds3/pull/8/commits/fd840de234b8d695dbc901aea89a382b02cb4d20).
- The log has no matches.

Perhaps someone, somewhere knows. If I were to be incentivised, maybe I would dig deeper.
Imagine if there was a powerful weapon in [Dark Forest](https://zkga.me/), that
could only be activated by providing the solution to a set of hashes?

Once known, I could upload the solution to the database, and all that piece of the puzzle
would be solved forever. Just how many contracts are there, whose names are lost to time
and attrition. What fraction of those are irrevocably lost and what fraction are hidden on
private repositories or personal folders. What if the list made and published, and those who
conquered more hashes were rewarded with black-matter partial-rift cannon shots? Perhaps
those missing hashes would magically appear over a weekend.

Now that I think about it, I do have access to the ABI for that contract. Submitting
that to 4bytes yields:

`Found 21 function and event signatures. Imported 4, Ignored 0, Skipped 17 duplicates.`

Now the database has one new function signature entry that matches the original search:

```
ID	Text Signature	Bytes Signature
192934	updateProxyTo(address)	0xfd840de2
```

Looking in the Event Signatures section, I now see a match, though I am unsure if it was
already there and have only found it now.

```
ID	Text Signature	Hex Signature
978	ProxyUpdated(address,address)	0xd32d24edea94f55e932d9a008afc425a8561462d1b1f57bc6e508e9a6b9509e1
```

At any rate, now there is a complete solution to the puzzle. Isn't that interesting.

Thanks goes, as always, to the people working to make a more robust open world.