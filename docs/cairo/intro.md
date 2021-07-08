---
layout: page
title:  "Cairo"
permalink: /cairo/intro/
toc: true
---

Cairo is programming language that can be used to write blockchain applications.
The language is novel in that it converts program logic into STARK proofs. This
enables an extremely favourable increase in the throughput of a given application.

- Learn [**Cairo by example**]({{ site.baseurl }}{% link cairo/by_example.md %})

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

More topics?

- Learn what it means to [**build an application**]({{ site.baseurl }}{% link cairo/examples/solidity_skeleton.md %})
- Learn about [**different frameworks**]({{ site.baseurl }}{% link cairo/frameworks.md %})
- Examine different [**deployment options**]({{ site.baseurl }}{% link cairo/deployment_options.md %})
- Learn more about [**proofs**]({{ site.baseurl }}{% link cairo/proof_size.md %}).
- Follow a walkthrough of an [**AMM program deployed to Ropsten with SHARP**]({{ site.baseurl }}{% link cairo/amm_demo.md %}).
- Cover the basics. What is Cairo, and why does it matter? [**Level I**]({{ site.baseurl }}{% link cairo/descriptive_summary.md %}) and [**Level II**]({{ site.baseurl }}{% link cairo/technical_summary.md %})

Cairo was invented at [Starkware](https://www.cairo-lang.org/), and for complete and
accurate information on how to use the language, see
the [official docs](https://www.cairo-lang.org/docs/).
