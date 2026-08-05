---
layout: post
title: "The detection field built its replacements and never switched"
date: 2026-08-05
description: "The benchmarks the field reports numbers on were built 2022-2024. Newer ones exist and carry single-digit citation counts."
---

I spent this week building an evaluation harness for a prose rubric, and the corpus made a liar out of it.

The rubric flags constructions that mark machine-written text. Eighteen gates, thresholds calibrated against 2,602 human documents so the false-positive rate is a measured 5% rather than a guess. Then I validated it against HC3, the standard paired human/machine corpus, and nine of the eighteen gates fired *more often on human writing than on machine writing*. The two constructions any reader in 2026 notices first, the em dash and `not just X, it's Y`, scored 0.0x and 0.3x. By the corpus, my best rules were my worst.

The rules aren't wrong. HC3 was collected between December 2022 and January 2023, from the ChatGPT web interface. The paper never states a collection date; you infer it from the launch on 30 November 2022 and the arXiv posting on 18 January 2023. Those models did not write the way current models write, so the corpus cannot contain the behaviour I was testing for.

That is a boring conclusion about one dataset. The interesting version is what happens when you check whether the rest of the field has the same problem.

## What the field actually reports numbers on

RAID is the standard. Ten million documents, eleven genres, twelve adversarial attacks, ACL 2024, and a live leaderboard at raid-bench.xyz. Its generations were produced in a fifteen-day window, 1 to 15 November 2023, against eleven models topping out at `gpt-4-0613`. That window is stated plainly in Appendix E.6 of the paper, which is more than most benchmarks tell you. There has been no v2. The COLING 2025 shared task re-used the November 2023 generations.

M4 generated in 2023. M4GT-Bench added LLaMA-2 and Jais in early 2024. SemEval-2024 Task 8 ran on the same lineage, and the COLING 2025 successor re-used those generations too. DetectRL's four generators are `gpt-3.5-turbo`, `chat-bison@002`, `claude-instant-1.2`, and Llama-2-70b-chat, which pins collection to roughly late 2023.

So the corpora the field reports numbers on were built between late 2022 and early 2024, against models whose ceiling is GPT-4 and Claude Instant.

## The replacements exist

Here is where my first draft was wrong, and where I'd have been caught inside five minutes.

Newer benchmarks are not missing. They are numerous. MIRAGE (September 2025) covers seventeen generators including o3-mini, Claude 3.7, DeepSeek-R1, and Grok-2. MAGA-Bench (January 2026) has 936,000 samples across Qwen3, DeepSeek-V3 and R1, and Gemini 2.0. TSM-Bench (May 2026) reports detection accuracy dropping 10 to 40 percent relative to prior benchmarks. DetectRL-X (May 2026) is a direct refresh of DetectRL across eight languages with GPT-4o, Gemini 2.5, DeepSeek-V3, and Qwen-Max. AITDNA (June 2026), from UKP Lab, samples GPT-5.2, Gemini 3 Flash, Llama 4 Scout, and DeepSeek V3.2, and records the full human editing history rather than treating authorship as binary. OpAI-Bench (June 2026) runs on GPT-5.4.

The work is done. It is public. It uses current models.

Now the citation counts, checked on Semantic Scholar today. HC3: 892. M4: 190. MAGE: 184. RAID: 181. M4GT: 71. DetectRL: 48.

MIRAGE: 7. AITDNA: 2. MAGA-Bench: 1. TSM-Bench: 1. DetectRL-X: 0.

Some of that is just age; a June 2026 paper has not had time to be cited. But the gap is three orders of magnitude, and the older corpora are still what new detector papers benchmark against. The field built the replacements and has not switched to them.

## The measurement nobody has made

I went looking for the obvious study: train a detector on 2022-era output, then plot its accuracy against generator vintage, 2022 through 2026. One independent variable, one curve.

It does not exist.

There is a great deal of adjacent work, and it is good work. "Rethinking AI-Generated Text Detection" (July 2026) shows a plain fine-tuned RoBERTa matches specialised detectors in-distribution and degrades sharply when the topic or the generating model changes, and that more training data does not close the gap. "Hitting a Moving Target" (June 2026) names three post-deployment shifts explicitly, including new LLM releases, and reports that commercial Pangram catches 24.1 percent of their adversarial humanised text against 90.5 percent for a test-time-adaptation approach. "Base Models Look Human To AI Detectors" (May 2026) finds GPTZero and Pangram rate base-model output as overwhelmingly human while flagging the instruction-tuned versions of the same models, and concludes that detectors are tracking artifacts of instruction tuning rather than any invariant property of machine text. "Spotlights and Blindspots" (April 2026) runs fifteen detectors and finds the rank order flips depending on which dataset and metric you pick.

Every one of those frames the problem as cross-generator generalisation or distribution shift. None isolates release date. The difference matters, because "detector fails on an unseen model" and "detector fails on models released after its training corpus" are different claims with different remedies. The first is solved by broader coverage. The second is only solved by refresh cadence, and no widely-used benchmark has one.

## What a rule looks like when the corpus can see it

The em dash is the one construction where somebody has done the work properly.

A pre-registered study posted in June 2026 measured em-dash prevalence in the Discussion sections of 69,632 first-version medRxiv preprints from 2020 to 2025. Prevalence rose from 4.23 percent before 30 November 2022 to 11.58 percent after, an absolute increase of 7.35 points with a confidence interval of 6.94 to 7.77. It was not a jump at launch. It was roughly 4 percent through 2023, 8.0 percent in 2024, and 20.3 percent in 2025. A placebo split inside the pre-LLM era moved the number by 0.13 points, which is nothing. The analysis plan was frozen on OSF before any confirmatory result.

The authors are careful, and their caveat belongs here: the em dash is a population-level indicator, not a per-paper detector, and the design cannot establish causality.

Read that timeline against HC3's collection window. The em-dash rise was still four years from its 2025 peak when HC3 was collected. My gate scoring 0.0x on that corpus is not evidence about the rule. It is a date stamp.

There is a second finding that cuts the other way and deserves equal billing. "The Last Fingerprint" (March 2026) tested twelve models across five providers under a markdown-suppression experiment and found em-dash rates ranging from 0.0 per thousand words, literally zero, for Meta's Llama models, to 9.1 for GPT-4.1. The paper argues the em dash is markdown leaking into prose, which makes it a fingerprint of a fine-tuning procedure rather than a property of machine writing in general. A rule that catches OpenAI models and misses Meta's is not a detector. It is a vendor classifier.

For `not just X, it's Y`, the construction I would have bet on hardest, there is no quantitative study at all. I searched arXiv for negative parallelism, antithesis, and contrastive-construction framings and found nothing. The only figures in circulation come from journalism I could not open directly, so I am not repeating them. The most recognisable tic in current model prose has never been measured, and that is its own small indictment.

## What I think follows

Three things, in the order I'd act on them.

Cite the benchmark's generation date, not its publication date. RAID states its window in an appendix. Most do not, and the gap between generation and publication is routinely a year.

Report per-generator results and treat the newest model in the corpus as the ceiling of what your number describes. A detector at 0.95 AUROC on RAID has demonstrated something about `gpt-4-0613`. That may still be useful. It is not a claim about 2026.

Somebody should run the vintage study. Fixed detector, fixed human corpus, machine text stratified by model release date, accuracy plotted against vintage. It is a week of API calls and a plot. Its absence is the reason nobody can say whether detection is getting harder or just differently hard.

I am building the corpus for it, which means I have an interest in the answer and you should discount accordingly. The licensing constraints are the hard part, not the generation, and I'll write that up separately.

---

**Sources.** RAID: [arXiv:2405.07940](https://arxiv.org/abs/2405.07940), leaderboard [raid-bench.xyz](https://raid-bench.xyz/). HC3: [arXiv:2301.07597](https://arxiv.org/abs/2301.07597). M4: [arXiv:2305.14902](https://arxiv.org/abs/2305.14902). M4GT-Bench: [arXiv:2402.11175](https://arxiv.org/abs/2402.11175). SemEval-2024 Task 8: [arXiv:2404.14183](https://arxiv.org/abs/2404.14183). DetectRL: [arXiv:2410.23746](https://arxiv.org/abs/2410.23746). MIRAGE: [arXiv:2509.14268](https://arxiv.org/abs/2509.14268). MAGA-Bench: [arXiv:2601.04633](https://arxiv.org/abs/2601.04633). TSM-Bench: [arXiv:2605.31113](https://arxiv.org/abs/2605.31113). DetectRL-X: [arXiv:2605.15518](https://arxiv.org/abs/2605.15518). AITDNA: [arXiv:2606.04906](https://arxiv.org/abs/2606.04906). OpAI-Bench: [arXiv:2606.06481](https://arxiv.org/abs/2606.06481). Distribution shift: [arXiv:2607.03680](https://arxiv.org/abs/2607.03680). Moving target: [arXiv:2606.25152](https://arxiv.org/abs/2606.25152). Base models: [arXiv:2605.19516](https://arxiv.org/abs/2605.19516). Spotlights and blindspots: [arXiv:2604.16607](https://arxiv.org/abs/2604.16607). Em-dash study: [arXiv:2606.29540](https://arxiv.org/abs/2606.29540), pre-registration OSF HFT8C. Markdown fingerprint: [arXiv:2603.27006](https://arxiv.org/abs/2603.27006). Citation counts from Semantic Scholar, 5 August 2026.

The harness and the rubric are at [prose-eval](https://github.com/wolfvswhale/prose-eval) and [bluepencil](https://github.com/wolfvswhale/bluepencil), including the results that made both look bad.
