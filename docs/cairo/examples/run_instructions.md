---
layout: page
title:  "Run instructions"
permalink: /cairo/examples/run_instructions/
toc: false
---

These are the steps to follow to run a given Cairo program (`program.cairo`) with an
associated input file (`input.json`).

### Initial Setup

One-time only. If this step has been completed, begin from "Activate environment".

Cairo is python based. An easy way to get started is:

Navigate to a directory where you would like to install Cairo and create and
activate a virtual environment called `cairo_venv`.

```
python3 -m cairo_venv .
source cairo_venv/bin/activate
```

Install `cairo-lang` using pip.

```
pip3 install cairo-lang
```

For troubleshooting, see the [official docs](https://www.cairo-lang.org/docs/quickstart.html).

### Activate environment

Activate the python virtual environment `cairo_venv`.

```
source cairo_venv/bin/activate
```

### Compile code

Compile the code to produce and output called `compiled_program.json`.

```
cairo-compile program.cairo --output compiled_program.json
```

### Run code

```
cairo-run --program=compiled_program.json --program_input=input.json \
    --print_output --layout=small --tracer
```

### Send the code for proving

This instruction sends the `program.cairo` and `input.json` to the SHARP
proving service.

```
cairo-sharp submit --source program.cairo --program_input input.json
```

### Interact with proof

You can navigate to the on-chain verifier contract.

The contract can be queried with a read method `isValid`. Passing the fact hash
will return `True` once the SHARP service has created the proof, bundled it, and
sent it to the contract.

### Using the proof

You can deploy a smart contract which can integrate the proof as part of a larger
application. More details can be found [here](solidity_skeleton.md).