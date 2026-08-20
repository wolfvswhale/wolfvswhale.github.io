---
layout: post
title: "I ran the vintage study. Release date does not predict detectability."
date: 2026-08-05
description: "Eighteen generators, 2019 to 2025. Release date does not predict detection rate. Instruction tuning does."
---

In the last review I complained that nobody had measured the obvious thing: take a detector, plot its accuracy against the release date of the model that produced the text, and see whether the curve falls. I said it was a week of API calls and a plot.

I was wrong about the API calls. The data already exists. RAID labels every generation with the model that made it, spanning GPT-2 through `gpt-4-0613`. MAGA-Bench extends the range to May 2025. Attach a public release date to each generator, cut the results that way, and the curve draws itself.

Here it is, across 18 generators and 15,500 documents.

![Detection rate against generator release date, and the base-versus-chat comparison]({{ site.baseurl }}/assets/vintage.png)

The left panel is the study I asked for. The right panel is what actually explains the numbers.

## The method, briefly

Train on generators released before July 2023, which simulates someone building a detector when those were the newest models available. Then fix the score threshold so exactly 5% of held-out human documents are flagged, and measure per-generator recall at that fixed false-positive rate.

Fixing the threshold on the human class matters. Report raw accuracy instead and a detector can look better on a generator merely by becoming more trigger-happy. Human documents have no vintage, so the same held-out human pool serves every generator and any movement in the curve has to come from the machine side.

Three detectors, chosen to have different inductive biases: character n-gram TF-IDF, word n-gram TF-IDF, and a 38-feature interpretable model built from sentence-length variance, paragraph shape, punctuation rates, and lexical diversity. If an effect is real it should show up in all three.

## Release date does not predict detectability

Within RAID, where corpus and genre are held constant, the rank correlation between release date and detection rate is **positive**: +0.31, +0.34, +0.42.

Newer generators are more detectable, not less. GPT-2 from 2019 sits at 29.5%. GPT-4 from June 2023 sits at 74.7%. That is the opposite of the assumption the whole field operates on.

Pool RAID with MAGA-Bench and the correlation flips to −0.26 and −0.55 for the two n-gram detectors. That looks like the decay everyone expects, and it is an artifact: every post-2024 generator in this study lives in MAGA, so the pooled correlation is measuring the corpus boundary, not the passage of time. The dashed line in the figure is where the corpus changes. Most of the apparent slope is that line.

## Instruction tuning does

RAID contains three families where a base model and its instruction-tuned sibling shipped on the same day. That holds release date, family, corpus, and detector constant and varies only the tuning.

| family | base | instruction-tuned | difference |
|---|---|---|---|
| MPT | 42.4% | 73.6% | +31.3 |
| Cohere | 31.6% | 49.4% | +17.8 |
| Mistral | 31.0% | 69.5% | +38.4 |

Three for three, in the same direction, averaging 26 to 29 points across the n-gram detectors. Nothing in the date column moves recall that far.

This is not a new idea, and I want to be clear that I did not get there first. "Base Models Look Human To AI Detectors" reached the same conclusion in May 2026 from Llama-3 and Qwen-3 with a different method, and concluded that detectors track artifacts of instruction tuning rather than any invariant property of machine text. What this adds is an independent replication on older, different models, arriving from a study that was designed to look for something else.

The practical reading: when a detector is described as failing on a newer model, check whether it is failing on a *less instruction-tuned* one. Those are different problems.

## The result I did not expect

The three detectors disagree, and the disagreement is the most useful thing here.

| detector | RAID mean | MAGA mean | gap |
|---|---|---|---|
| char TF-IDF | 55.4% | 30.0% | −25.4 |
| word TF-IDF | 51.6% | 18.9% | −32.7 |
| style features | 50.2% | 53.3% | **+3.0** |

Both n-gram detectors lose roughly half their recall when the corpus changes. The interpretable style-feature detector does not move.

The same split shows up in the instruction-tuning comparison: +29 and +26 points for the n-gram detectors against +7.6 for the style features. Surface features are reading tuning artifacts and corpus idiosyncrasies. Structural features are reading something more stable.

If that generalises, a good deal of what the literature calls detector degradation is surface-feature brittleness rather than a property of newer models, and the fix is different from the one people are pursuing. More training data across more generators will not repair an n-gram model that is keying on the wrong thing.

I would not lean hard on this yet. It is three detectors, all linear, all trained on one corpus, and the style-feature model is my own so I have every incentive to like the result. It wants replication by someone who does not.

## What this does not show

**Vintage and capability are entangled and this design cannot separate them.** GPT-2 is both old and small; a 2025 model is both new and better. Some of the flat curve may be two effects cancelling out. Separating them needs same-capability models from different years, which mostly do not exist.

**The MAGA comparison is confounded** in exactly the way I accused the field of ignoring. Corpus, genre, and prompt construction all change at the same boundary as the date. Only the within-RAID correlation isolates date, and it rests on eleven generators.

**Release dates are contestable.** They are announcement dates for the specific checkpoint each corpus names, and the mapping is in the repo so it can be argued with rather than trusted. RAID's paper and its README disagree about whether its GPT-3 is `text-davinci-002` or `-003`; I used the paper.

**The newest generator here is May 2025.** That is the newest with per-generator labels in a public paired corpus, which is the original complaint biting me. Extending the curve into 2026 requires generation rather than reuse, and that is the next thing I am building.

## What I would change about how this gets reported

The question "does detection get harder over time" is badly formed and I asked it badly. Time is not a mechanism. Instruction tuning is a mechanism. Corpus construction is a mechanism. Model scale is a mechanism. Release date is a proxy that bundles all three and then hides which one moved.

A per-generator table with release dates, tuning status, and corpus attached costs nothing to produce and would make results like this legible at a glance. Most detection papers report a single pooled number.

Code, data-building script, results, and the figure are at [github.com/wolfvswhale/vintage-study](https://github.com/wolfvswhale/vintage-study), including the twenty-one tests and the controls that did not find anything.

---

**Sources.** RAID: [arXiv:2405.07940](https://arxiv.org/abs/2405.07940), ACL 2024, generations produced 1–15 November 2023. MAGA-Bench: [arXiv:2601.04633](https://arxiv.org/abs/2601.04633). Base models look human: [arXiv:2605.19516](https://arxiv.org/abs/2605.19516). Style features from [prose-eval](https://github.com/wolfvswhale/prose-eval).


## Changelog

**Version 1.0, 5 August 2026.** First published.
