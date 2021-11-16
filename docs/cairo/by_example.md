---
layout: page
title:  "Cairo by example"
permalink: /cairo/by-example/
toc: false
---


- First [setup using Nile]({{ site.baseurl }}{% link cairo/nile.md %}) which
makes managing files and command line calls to StarkNet simple.
- Second have a look at how [pytest works]({{ site.baseurl }}{% link cairo/pytest.md %})
- After you have set everything up, to run the examples:
    - Open two terminals in the project folder and run `source env/bin/activate`
    - In one run `nile node` to start a local devnet.
    - In the other run the commands in the examples.

Examples of different elements in Cairo and StarkNet.

- [Hello, World!]({{ site.baseurl }}{% link cairo/examples/hello_world.md %})
- [First application]({{ site.baseurl }}{% link cairo/examples/first_application.md %})
- [Array]({{ site.baseurl }}{% link cairo/examples/array.md %})
- [Data types]({{ site.baseurl }}{% link cairo/examples/data_types.md %})
- [Variables]({{ site.baseurl }}{% link cairo/examples/variables.md %})
- [Read and write values]({{ site.baseurl }}{% link cairo/examples/read_write.md %})
- [Read and write tuples]({{ site.baseurl }}{% link cairo/examples/read_write_tuple.md %})
- [Currency]({{ site.baseurl }}{% link cairo/examples/currency.md %})
- [Constructor]({{ site.baseurl }}{% link cairo/examples/constructor.md %})
- [Custom imports]({{ site.baseurl }}{% link cairo/examples/custom_imports.md %})
- [Contract calls]({{ site.baseurl }}{% link cairo/examples/contract_calls.md %})
- [Array argument]({{ site.baseurl }}{% link cairo/examples/array_argument.md %})
- [Array returns]({{ site.baseurl }}{% link cairo/examples/array_returns.md %})
- [Struct returns]({{ site.baseurl }}{% link cairo/examples/struct_returns.md %})

# YMMV Examples

These examples have core principles, but have not been tested with
`cairo-lang version >0.5.0` and may not compile. I will move them to the
section above after checking them and adding nile-based notes.

- [If-Else]({{ site.baseurl }}{% link cairo/examples/if_else.md %})
- [Loop]({{ site.baseurl }}{% link cairo/examples/loop.md %})
- [Mapping]({{ site.baseurl }}{% link cairo/examples/mapping.md %})
- [Tuple]({{ site.baseurl }}{% link cairo/examples/tuple.md %})
- [Struct]({{ site.baseurl }}{% link cairo/examples/struct.md %})
- [Pointer]({{ site.baseurl }}{% link cairo/examples/pointer.md %})
- [Data locations]({{ site.baseurl }}{% link cairo/examples/data_locations.md %})
- [Function]({{ site.baseurl }}{% link cairo/examples/function.md %})
- [Assert]({{ site.baseurl }}{% link cairo/examples/assert.md %})
- [Generate message]({{ site.baseurl }}{% link cairo/examples/generate_message.md %})
- [Message L1]({{ site.baseurl }}{% link cairo/examples/message_L1.md %})
- [Receive message]({{ site.baseurl }}{% link cairo/examples/receive_message.md %})
- [Pedersen hash]({{ site.baseurl }}{% link cairo/examples/pedersen_hash.md %})
- [Verify ECDSA signature]({{ site.baseurl }}{% link cairo/examples/verify_ECDSA.md %})
- [Math]({{ site.baseurl }}{% link cairo/examples/math.md %})
- [Math comparison operators]({{ site.baseurl }}{% link cairo/examples/math_comparison.md %})
- [Bitwise operations]({{ site.baseurl }}{% link cairo/examples/bitwise.md %})
- [Dictionary]({{ site.baseurl }}{% link cairo/examples/default_dict.md %})
