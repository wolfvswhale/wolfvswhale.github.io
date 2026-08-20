---
layout: page
title: About
permalink: /about/
---

I review how AI benchmarks are graded, and I build the measurement tools underneath that work.

I came to evaluation sideways. I had a house editing standard for stripping machine tics out of prose, wanted to know whether it actually worked, and built a harness to find out. The answer was partly no, which turned out to be the interesting part. Before that, four years producing court transcripts, each one checkable against the original recording.

**[gdpval-audit](https://github.com/wolfvswhale/gdpval-audit)** opens all 10,453 rubric criteria in GDPval and asks whether the grading is uniform across the 220 tasks. It is not. Maximum attainable score ranges from 22 to 234 points, one ungradeable line appears in 138 tasks, and four tasks have no scoring floor. [Read the review]({{ site.baseurl }}/2026/08/06/gdpval-rubrics.html).

**[vintage-study](https://github.com/wolfvswhale/vintage-study)** asks whether detectors degrade as generators get newer. They don't. Instruction tuning predicts detectability; release date doesn't. [Read the review]({{ site.baseurl }}/2026/08/05/vintage-study.html).

**[prose-eval](https://github.com/wolfvswhale/prose-eval)** tests whether a hand-written editorial rubric can separate machine-written prose from human. Structural cadence carries nearly all the signal; the fitted model transfers below chance to a new domain.

**[bluepencil](https://github.com/wolfvswhale/bluepencil)** is the tool that harness justifies. Eighteen gates, every threshold calibrated against 2,602 human documents so the false-positive rate is measured at 5%.

**[prose-cadence-stats](https://huggingface.co/datasets/wolfvswhale/prose-cadence-stats)** publishes the measurements underneath all of it, so anyone can recompute a threshold or challenge one.

Every review carries a version number and a changelog.

[GitHub](https://github.com/wolfvswhale) · [Hugging Face](https://huggingface.co/wolfvswhale) · [LinkedIn](https://www.linkedin.com/in/jaldermanlyell/)
