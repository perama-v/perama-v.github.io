---
layout: page
title:  "Cairo"
permalink: /cairo/intro/
toc: true
---

Cairo is programming language that can be used to write blockchain applications.
The language is novel in that it converts program logic into STARK proofs. This
enables an extremely favourable increase in the throughput of a given application.

- Learn [**Cairo by example**]({{ site.baseurl }}{% link cairo/by_example.md %}).
- Explore modules from the
[**common library**]({{ site.baseurl }}{% link cairo/cairo_common_lib.md %}).
- Understand how accounts work in StarkNet, with **account abstraction**
in a
[first]({{ site.baseurl }}{% link cairo/account_abstraction.md %}) and
[second]({{ site.baseurl }}{% link cairo/account_abstraction_test.md %}) exploration.


The benefit of STARKs can be summarised in this python pseudocode.

{% highlight python %}
    # This is not Cairo code.

    from mainnet import security

    def starkify(transactions):
        calldata = get_proof(transactions)
        return calldata

    batch = starkify('100x_more_txs')

    # Prints publicly verifiable transactions at fraction of the cost.
    print(batch)
{% endhighlight %}

Some topics:

- Write StarkNet contract [unit tests]({{ site.baseurl }}{% link cairo/examples/unit_test.md %}).
- Compare and contrast:
    - Cairo [programs](examples/building_blocks/skeleton/program_sharp.md), for creating provable facts on L1.
    - Cairo [contracts](examples/building_blocks/skeleton/program_starknet.md), for creating composable L2 applications. They are built from Cairo programs.
- Use the StarkNet [command line interface]({{ site.baseurl }}{% link cairo/starknet_cli.md %})
- Review the StarkNet roadmap [here]({{ site.baseurl }}{% link cairo/deployment_options.md %})
- Learn more about [proofs]({{ site.baseurl }}{% link cairo/proof_size.md %}).
- Cover the basics. What is Cairo, and why does it matter?
    - [Introductory]({{ site.baseurl }}{% link cairo/descriptive_summary.md %})
    - [More detail]({{ site.baseurl }}{% link cairo/technical_summary.md %})
- Explore what [ENS on StarkNet]({{ site.baseurl }}{% link cairo/ens/bridge.md %}) might look like
- Use an L1 solidity contract to ingest an arbitrary Cairo program proof [here]({{ site.baseurl }}{% link cairo/examples/solidity_skeleton.md %}).
- Follow a walkthrough of an AMM program deployed to Ropsten with SHARP [here]({{ site.baseurl }}{% link cairo/amm_demo.md %}).
- Build a game on L2 [here]({{ site.baseurl }}{% link cairo/game/world_design.md %}).

Cairo was invented at [Starkware](https://www.cairo-lang.org/), and for complete and
accurate information on how to use the language, see
the [official docs](https://www.cairo-lang.org/docs/).
