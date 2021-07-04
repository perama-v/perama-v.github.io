---
layout: page
title:  "Cairo"
permalink: /cairo/intro/
toc: true
---

Cairo is programming language that can be used to write blockchain applications.
The language is novel in that it converts program logic into STARK proofs. This
enables an extremely favourable increase in the throughput of a given application.

The benefit of STARKs can be summarised in this python pseudocode.

{% highlight python %}

    from mainnet import security

    def starkify(transactions):
        calldata = get_proof(transactions)
        return calldata

    batch = starkify('100x_more_txs')

    # Prints publicly verifiable transactions at fraction of the cost.
    print(batch)
{% endhighlight %}

How to use Cairo?
- Check out some [**building blocks**]({{ site.baseurl }}{% link cairo/building_blocks.md %})
- Look at library [**modules**]({{ site.baseurl }}{% link cairo/cairo_common_lib.md %}) that can be imported.
- [**Build an application**]({{ site.baseurl }}{% link cairo/examples/solidity_skeleton.md %})
- Examine different [**deployment options**]({{ site.baseurl }}{% link cairo/deployment_options.md %})
- Understand [**proof sizes**]({{ site.baseurl }}{% link cairo/proof_size.md %}).
- Follow a walkthrough of an [**AMM deployment**]({{ site.baseurl }}{% link cairo/amm_demo.md %}).

The Cairo rabbit hole goes deep.

For fast explanation what Cairo is, from the foundation up, read this
[**written description**]({{ site.baseurl }}{% link cairo/descriptive_summary.md %})

To dive a little deeper into some technical details along the way, read this
more [**technical description**]({{ site.baseurl }}{% link cairo/technical_summary.md %})

Cairo was invented at [Starkware](https://www.cairo-lang.org/), and for complete and
accurate information on how to use the language, see
the [official docs](https://www.cairo-lang.org/docs/).
