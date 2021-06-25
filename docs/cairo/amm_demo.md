---
layout: page
title:  "Automated Market Maker Demo"
permalink: /cairo/amm-demo/
toc: false
---

## Part I

Deploying a Cairo-cased AMM, Variation II, Part I.

Automated market makers (AMM) are a relatively familiar Ethereum contract. The Cairo docs
have a
[tutorial](https://www.cairo-lang.org/build-a-scalable-cairo-basesd-automated-market-maker/)
showing the deployment of an AMM written in Cairo, which here is called "Variation I".

This page follows along with that example,
using Hardhat for contract deployment and contract interaction. This "Variation II"
might be useful for comparison, and for learning how to integrate Cairo into your toolset
if you already use Hardhat. Those unfamiliar with Hardhat are recommended to follow
along and give it a try.

### **Tools**

- VS Code Solidity and Cairo extensions (linting)
- Hardhat command line (Solidity contract development)
- Cairo-lang command line (Cairo program development)
- Personal Ropsten node, Infura or Alchemy (Ropsten network interaction)

### **Flow**

This an overview of the flow for deployment.

Cairo **program creation** steps. This is where design and initial deployment of the Cairo program
happens. The program will accept user input, which will be synthesised into a proof and stored
as a fact on-chain.
```
program.cairo  # Copy the program code.
  -> input.json  # Copy the program input file.
    -> pip3 install cairo-lang  # Get Cairo tooling set up.
      -> cairo-sharp submit  # Send it to SHARP service.
        -> verifyAndRegister  # (passive) await Ropsten proof.
```
Solidity **contract deployment** steps. This is where the design and deployment of a contract
happens. The contract will be speaking with the already-live verifier contract to feed the
AMM system with funds and store final balances.
```
amm.sol  # Copy the contract code.
  -> Configure Hardhat  # Solidity versions, etc.
    -> Create contract constructor arguments  # Initial values.
      -> hardhat run scripts/deploy.js  # Prepare & deploy locally.
        -> Faucet hunting  #  Get some Ropsten ether.
          -> run --network ropsten  # Deploy to real testnet.
```
Solidity **contract interaction** steps. This is where a solidity contract is designed to utilise
the facts that the Cairo program generated. This transaction converts facts to state changes.
```
amm.sol
  -> design update-amm.js  # Decide what values to send to L1 AMM.
    -> hardhat run scripts/update-amm.js  # Run interaction script.
```
Cairo **program interaction** steps. This is where a user creates a new trade by specifying new
inputs to the program, which is unmodified from the initial design. The new inputs are converted
into facts that will be used by the Solidity contract to change the ownership of tokens.
```
program.cairo  # Use existing program code.
  -> input.json  # Decide on new inputs as a "trade".
    -> cairo-sharp submit  # Send the trade for proving
      -> verifyAndRegister  # (passive) await Ropsten proof.
```

### **First set up Cairo**

Follow these [instructions]({{ site.baseurl }}{% link cairo/examples/run_instructions.md %})

### **Create program.cairo**

Make a file called `program.cairo` and populate it with
[this Cairo program](https://github.com/starkware-libs/cairo-lang/blob/master/src/demo/amm_demo/amm.cairo)

This program will take one token and swap it for another, as an automated market maker.

### **Hash program**

Calculate the hash of the compiled program:
```
cairo-compile program.cairo --output=compiled_program.json
cairo-hash-program --program=compiled_program.json
```
Verify that it matches:
```
0x594483e1afa95f330c44d858fef134551766e73cf2ecb7dab915fcf36435a21
```

### **Extensions**

Install the VS Code
[solidity extension](https://marketplace.visualstudio.com/items?itemName=JuanBlanco.solidity).

Install the VS Code Cairo extension using instructions
[here](https://www.cairo-lang.org/docs/quickstart.html#visual-studio-code-setup).

### **Hardhat setup**

Install hardhat with these steps, or follow the
[official instructions](https://hardhat.org/getting-started/#installation).

```
# Install npm from https://nodejs.org/en/download/
# Make a new project folder, npm init and accept defaults.
npm init
# Install hardhat and useful packages.
npm install --save-dev hardhat
npm install --save-dev @nomiclabs/hardhat-waffle ethereum-waffle chai
npm install --save-dev @nomiclabs/hardhat-ethers ethers
# Create a project ("Create a sample project")
npx hardhat
```

That will create:

- `hardhat.config.js`, a file where configurations are defined.
- `scripts/`, a folder for .js scripts that are used to deployand interact with contracts.
- `contracts/`, a folder for solidity contracts.

Now create following empty files:

- `secrets.json`, a configuration for keeping API keys and mnemonics separate.
- `contracts/AmmDemo.sol`, a solidity contract for interacting with amm proofs.
- `scripts/amm-deploy.js`, a script for deploying the contract either locally or to a network.
- `scripts/amm-interact.js`, a script to interact with a deployed contract.

### hardhat.config.js

Add Solidity compiler `v0.5.2` for the amm.sol contract to use.
```
require("@nomiclabs/hardhat-waffle");

const { infuraApiKey, mnemonic } = require('./secrets.json');

module.exports = {
  solidity: {
    compilers: [
      {
        version: "0.8.3"
      },
      {
        version: "0.7.3"
      },
      {
        version: "0.5.2"
      }
    ]
  },
  networks: {
    ropsten: {
      url: `${infuraApiKey}`,
      accounts: {mnemonic: mnemonic}
    }
  }
}
```

### AmmDemo.sol

Copy the contract below in into `contracts/AmmDemo.sol`. Note the convention, to make
the contract filename and the contract name in the code identical.
This contract is from
[here](https://github.com/starkware-libs/cairo-lang/blob/master/src/demo/amm_demo/amm_contract.sol)

Note that the heart of this contract is ``accountTreeRoot``, the root of a Merkle tree
that is used to represent the state of a system. In this case, that system is a market maket
with two tokens, A and B. The contract keeps the actual total balance of each token,
where on the Cairo-based L2, these tokens are owned by many different actors.
An L2 actor may claim their token on L1 by using their balance and some other Merkle branches.

The L2->L1 withdrawal component of the AMM is not implemented in this example, but would
involve a Cairo program executing a withdrawl operation. The withdrawl proof, which
contains the new AMM account merkle root, would be stored by the Verifier contract.
The user could then trigger a withdrawl from the the AmmDemo contract, which would
accept the new Merkle root after checking that the proof for the withdrawl was verified.

```
// SPDX-License-Identifier: Cairo Program License (Source Available)
// Version 1.0, November 2020.
pragma solidity ^0.5.2;

contract IFactRegistry {
    /*
      Returns true if the given fact was previously registered in the contract.
    */
    function isValid(bytes32 fact)
        external view
        returns(bool);
}

/*
  AMM demo contract.
  Maintains the AMM system state hash.
*/
contract AmmDemo {
    // Off-chain state attributes.
    uint256 accountTreeRoot_;

    // On-chain tokens balances.
    uint256 amountTokenA_;
    uint256 amountTokenB_;

    // The Cairo program hash.
    uint256 cairoProgramHash_;

    // The Cairo verifier.
    IFactRegistry cairoVerifier_;

    /*
      Initializes the contract state.
    */
    constructor(
        uint256 accountTreeRoot,
        uint256 amountTokenA,
        uint256 amountTokenB,
        uint256 cairoProgramHash,
        address cairoVerifier)
        public
    {
        accountTreeRoot_ = accountTreeRoot;
        amountTokenA_ = amountTokenA;
        amountTokenB_ = amountTokenB;
        cairoProgramHash_ = cairoProgramHash;
        cairoVerifier_ = IFactRegistry(cairoVerifier);
    }

    function updateState(uint256[] memory programOutput)
        public
    {
        // Ensure that a corresponding proof was verified.
        bytes32 outputHash = keccak256(abi.encodePacked(programOutput));
        bytes32 fact = keccak256(abi.encodePacked(cairoProgramHash_, outputHash));
        require(cairoVerifier_.isValid(fact), "MISSING_CAIRO_PROOF");

        // Ensure the output consistency with current system state.
        require(programOutput.length == 6, "INVALID_PROGRAM_OUTPUT");
        require(accountTreeRoot_ == programOutput[4],
            "ACCOUNT_TREE_ROOT_MISMATCH");
        require(amountTokenA_ == programOutput[0], "TOKEN_A_MISMATCH");
        require(amountTokenB_ == programOutput[1], "TOKEN_B_MISMATCH");

        // Update system state.
        accountTreeRoot_ = programOutput[5];
        amountTokenA_ = programOutput[2];
        amountTokenB_ = programOutput[3];
    }
}
```

Then run:

```npx hardhat compile```

This will compile all the contracts in `contracts/` directory, and will ensure that there are
no errors in each contract. The contracts are compiled with the `x.y.z` version
of the Solidity compiler as defined in the line at the top of the contract `.sol`
file, `pragma solidity ^x.y.z;`. A smart contract is designed for a particular compiler
version, and using another version may introduce a compilation error or change in contract
behaviour.

### Contract deployment: Local Testnet

**Constructor arguments**

The original tutorial example deploys the contract with a
[custom python script](https://github.com/starkware-libs/cairo-lang/blob/master/src/demo/amm_demo/demo.py).
As part of deployment, the contract is initialised with a set of initial values, which are
passed to the constructor.

- **`account_tree_root`**.
  - Defined in the python script as `get_merkle_root(batch_prover.accounts)`.
  - A merkle tree root, representing the unique state of the system, see below for details.
  - `3262995978462033705189630496750790514901299827561453807468505774450708589253`
- **`amount_token_a`**, initial balance of token A in the AMM. An arbirary value.
  - Defined in the python script as `batch_prover.balance.a`.
  - Defined here as `100`.
- **`amount_token_b`**, initial balance of token B in the AMM. An arbirary value.
  - Defined in the python script as `batch_prover.balance.b`.
  - Defined here as `1000`.
- **`program_hash`**, program hash, calculated by the Cairo command line interface (above).
  - Defined in the python script as `compute_program_hash_chain(batch_prover.program)`.
  - `0x594483e1afa95f330c44d858fef134551766e73cf2ecb7dab915fcf36435a21`
- **`cairo_verifier`**, Ropsten address where the verifier is deployed.
  - Defined in the python script as `batch_prover.sharp_client.contract_client.contract.address`.
  - Current address `0x2886D2A190f00aA324Ac5BF5a5b90217121D5756` can be found in the
  [SHARP config file](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/sharp/config.json).

The `account_tree_root` is a part of a merkle tree. It represents the unique state of the
system. In the AMM Demo, the system is initialised with 5 accounts, each with a random
amount of token A and B. Tokens exist either in the AMM or in a user account.

So, where the StarkWare example generates batches for simulation, here a single state is
used. For every token A, the AMM has 10 token B. Thus for the first trade, approximately
10 of token B can purchased for every token A.

```
# Accounts ordered by index, with public key and balances A and B.
accounts_dictionary = {
    0: Account(0x1a, Balance(100, 50)),  # 100 A, 50 B.
    1: Account(0x3a, Balance(125, 500)),  # Public key 0x3a.
    2: Account(0xff, Balance(550, 200)),  # Account number 2.
    3: Account(0x7e, Balance(165, 40)),
    4: Account(0x33, Balance(750, 200))
}
```
This is the "state" of the system. The total tokens in circulation in the L2 are calculated
as the sum of the user balances plus the initial 100 token A and 1000 toke B in the contract.

The Merkle root is based on the user balances, and can be calculated with a series of
hash operations. This
[merkle tree guide]({{ site.baseurl }}{% link cairo/examples/merkle_trees.md %}) outlines
the steps by which this occurs, and exposes the code used to calculate the hash.

With the python cairo-lang environment activated, StarkWare's Merkle tree algorithm
can be imported easily.

```
import dataclasses
from typing import Dict
from starkware.cairo.common.small_merkle_tree import MerkleTree
from starkware.cairo.lang.vm.crypto import pedersen_hash

@dataclasses.dataclass
class Balance:
    """
    Represents the balance of each of the two tokens.
    """
    a: int
    b: int

@dataclasses.dataclass
class Account:
    pub_key: int
    balance: Balance

def get_merkle_root(accounts: Dict[int, Balance]) -> int:
    """
    Returns the merkle root given accounts state.
    accounts: the state of the accounts (the merkle tree leaves).
    """
    tree = MerkleTree(tree_height=10, default_leaf=0)
    return tree.compute_merkle_root([
        (i, pedersen_hash(
            pedersen_hash(a.pub_key, a.balance.a), a.balance.b))
        for i, a in accounts.items()])

accounts_dictionary = {
    0: Account(0x1a, Balance(100, 50)),
    1: Account(0x3a, Balance(125, 500)),
    2: Account(0xff, Balance(550, 200)),
    3: Account(0x7e, Balance(165, 40)),
    4: Account(0x33, Balance(750, 200))
}

account_tree_root = get_merkle_root(accounts_dictionary)
print(account_tree_root)

```
The Merkle root is:
`3262995978462033705189630496750790514901299827561453807468505774450708589253`

Once the hash has been obtained, the contract can be deployed. From then, all modifications
to the system happen as state updates triggered by Cairo program proofs, manifesting
as updates to the Merkle root.

**Deployment script**

Create a deployment script `scripts/amm-deploy.js` and populate it with the following:
```
const hre = require("hardhat");

const accountRoot = "3262995978462033705189630496750790514901299827561453807468505774450708589253";
const tokenA = 100;
const tokenB = 1000;
const programHash = "0x594483e1afa95f330c44d858fef134551766e73cf2ecb7dab915fcf36435a21";
const verifierAddress = "0x2886D2A190f00aA324Ac5BF5a5b90217121D5756";

async function main() {
    // Get the contract by name.
    const AmmContract = await hre.ethers.getContractFactory("AmmDemo");
    // Deploy it, passing important values to the constructor
    const amm = await AmmContract.deploy(
        accountRoot,
        tokenA,
        tokenB,
        programHash,
        verifierAddress);
    // Wait for deployment confirmation.
    await amm.deployed();
    // Return the address it is deployed to.
    console.log("amm deployed to:", amm.address);
}
main()
    .then(() => process.exit(0))
    .catch(error => {
        console.error(error);
        process.exit(1);
    });

```
Then run the script to deploy the contract to temporary local network:
```
npx hardhat run scripts/amm-deploy.js
```

The address of the AmmDemo contract be displayed in the console:
`amm deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3`.

**Summary**

That concludes Vairation II, part I.

Steps completed:
- Automated market maker was considered, where trades occur on layer 2,
and a STARK-based rollup stores state changes on-chain.
- A Cairo program was created and it's unique hash was recorded.
- A set of 5 dummy accounts were created, each with some of tokens A and B.
- A Merkle tree was constructed and its root recorded.
- A Solidity contract was deployed to a temporary Ethereum testnet.
- During deployment, the contract was initialised with:
  - The program hash.
  - The amount of each token in the AMM.
  - The Ropsten address of the live STARK proof Verifier contract.
  - The merkle root, representing the unique state of the 5 accounts.

## Part II

Deploying a Cairo-cased AMM, Variation II, Part II.

- The Solidity contract will be deployed to a local testnet.
- A transaction will be sent to try to update the state of the AMM.

### Contract deployment: Persistent local testnet

Before deploying to Ropsten, it will be nice to test interacting with
the contract. Skip straight to the Ropsten section if you like.

The steps will be:

1. Create the network.
2. Deploy the AMM.
3. Make a pretend fact registry contract and deploy it.
4. Send a transaction to the AMM to update the state.

Spin up a local persistent network with Hardhat.

```
npx hardhat node
```

**Local AMM deployment**

This network will watch for and integrate transactions sent by Hardhat.
Leave the network running and in another window, run the deployment scripts specifying the
network (`--network localhost`).

**Local verifier**

Make the file `contracts/PretendFactRegistry.sol`, populate it with the following:
```
// SPDX-License-Identifier: Cairo Program License (Source Available),
// Version 1.0, November 2020.
pragma solidity ^0.5.2;

contract PretendFactRegistry {

    constructor() public {}
    /*
      Returns true if the given fact was previously registered in the contract.
    */
    function isValid(bytes32 fact) external view returns (bool) {
        // Fact is evaluated, true/false returned.
        // This fake contract always returns true.
        return true;
    }
}
```
Make the file `scripts/verifier-deploy.js`, populate it with the following:

```
const hre = require("hardhat");

async function main() {
    // Get the contract by name.
    const VerifierContract = await hre.ethers.getContractFactory("PretendFactRegistry");
    // Deploy it.
    const verifier = await VerifierContract.deploy();
    // Wait for deployment confirmation.
    await verifier.deployed();
    // Return the address it is deployed to.
    console.log("verifier deployed to:", verifier.address);
}
main()
    .then(() => process.exit(0))
    .catch(error => {
        console.error(error);
        process.exit(1);
    });
```
Deploy it:

```
npx hardhat run scripts/verifier-deploy.js --network localhost
```

Note the the address:`verifier deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3`.

**Local AMM that trusts the pretend verifier**

The deployment script `scripts/amm-deploy.js` needs to contain the address of the pretend verifier.

```
const hre = require("hardhat");

const accountRoot = "3262995978462033705189630496750790514901299827561453807468505774450708589253";
const tokenA = 100;
const tokenB = 1000;
const programHash = "0x594483e1afa95f330c44d858fef134551766e73cf2ecb7dab915fcf36435a21";
const verifierAddress = "0x5FbDB2315678afecb367f032d93F642f64180aa3";

async function main() {
    // Get the contract by name.
    const AmmContract = await hre.ethers.getContractFactory("AmmDemo");
    // Deploy it, passing important values to the constructor
    const amm = await AmmContract.deploy(
        accountRoot,
        tokenA,
        tokenB,
        programHash,
        verifierAddress);
    // Wait for deployment confirmation.
    await amm.deployed();
    // Return the address it is deployed to.
    console.log("amm deployed to:", amm.address);
}
main()
    .then(() => process.exit(0))
    .catch(error => {
        console.error(error);
        process.exit(1);
    });

```

Deploy it:

```
npx hardhat run scripts/amm-deploy.js --network localhost
```
See that the network has registered the contract deployment. The contract may
now be called by another script. This mimics calling a function on a script
deployed on Ropsten.

**Local AMM state update**

Finally, test what it looks like to update the state of the AmmDemo.

Recall what the goal is for every update:

- Pass the Cairo program output to the AmmDemo contract.
- AMM contract computes the Fact (a hash based on the program hash
and the program outputs).
- Fact is sent to to the FactRegistry.
- If the Verifier has the Fact, return true.
- AMM contract process the program output, which consists of a 6 element array, corresponding to:
    - 0: Current Token A quantity in AMM.
    - 1: Current Token B quantity in AMM.
    - 2: New Token A quantity in AMM.
    - 3: New Token B quantity in AMM.
    - 4: Current account tree Merkle root.
    - 5: New account tree Merkle root.
- The AMM contract checks the old values match the stored values,
then updates them to the new values.

Recall that a pretend FactRegistry contract is used, which returns
`true` for every fact queried.

populate the ``scripts/amm-interact.js`` file with the following contents:

```
const hre = require("hardhat");

async function main() {
    const programOutput = [
        100,  // Current A.
        1000,  // Current B.
        110,  // New A.
        900,  // New B.
        "3262995978462033705189630496750790514901299827561453807468505774450708589253",  // Current Merkle root.
        "12345678"];  // New Merkle root (placeholder).

    // Get the contract details.
    const AmmContract = await hre.ethers.getContractFactory("AmmDemo");

    // Get deployed contract by its address.
    const amm = await AmmContract.attach(
        // The deployed contract address.
        "0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512"
    );

    // Call the function to store a new state.
    await amm.updateState(programOutput);
}
main()
    .then(() => process.exit(0))
    .catch(error => {
        console.error(error);
        process.exit(1);
    });

```

Interact with the contract using that script:

```
npx hardhat run scripts/amm-interact.js --network localhost
```

Observe that the local node processes the update state transaction.
A second execution of the above script fails with the error:

```
Error: VM Exception while processing transaction:
reverted with reason string 'ACCOUNT_TREE_ROOT_MISMATCH'
```

Which stems from the fact that the update script on the second call
no longer has function arguments that match the current state of the AMM contract.

The account root update in this example is left as a placeholder
value `12345678` for simplicity. Note that the AMM contract accepts the
fabricated state root in this example. If the verifier was operational,
it would have rejected a Fact representing an incorrectly computed root.
A proper state update will be calculated for the Ropsten testnet example below.

**Summary**

That concludes Vairation II, part II.

Steps completed:

- A local testnet was created that mines transactions produced by Hardhat.
- The AmmDemo.sol and a PretendFactRegistry.sol contracts were deployed.
- The AMM was updated was performed using a fabricated program output that was
verified by an "everything is true" verifier.
- The AMM succeeded in refusing a second update when program outputs in the submitted
update did not contain the current state of the contract.

## Part III (pending)

Deploying a Cairo-cased AMM, Variation II, Part III.

- The Solidity contract will be deployed to Ropsten.
- The Cairo program will be used to execute a L2 trades.
- The Solidity contract will be triggered to include updated post-trade state.

