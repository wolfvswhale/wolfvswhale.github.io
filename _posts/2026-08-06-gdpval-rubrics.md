---
layout: post
title: What GDPval's rubrics actually measure
date: 2026-08-06
description: 10,453 rubric criteria across 220 tasks. Score scales vary tenfold, one untestable line appears in 138 tasks, and four tasks have no scoring floor.
---

GDPval asks whether models can do economically valuable knowledge work. It contains 220 tasks drawn from 44 occupations, each with a prompt, supporting files, a reference deliverable produced by a working professional, and a grading rubric. It is one of the most cited benchmarks in the field, it appears in frontier model release announcements, and its dataset has been downloaded more than 120,000 times.

Epoch AI reviewed it in February 2026, alongside RLI and APEX-Agents, and made a structural argument: the tasks are performed in isolation from any broader business context, so a high score indicates task-level assistance rather than job automation. That is an argument about what the tasks are.

This is an argument about how they are graded. GDPval's score is the output of 10,453 written rubric criteria, and nobody has opened them. I did, and the finding is not that the rubrics are bad. Most of them are careful, and some are better than what I have seen inside professional audit shops. The finding is that they are wildly uneven, in specific ways that make GDPval's aggregate number less comparable across tasks than it looks.

## Main takeaways

- The maximum attainable score varies from **22 to 234 points** across the 220 tasks, a 10.6-fold spread, with 95 distinct values. Nothing in the published dataset normalizes this.
- One rubric line, **"Overall formatting and style of the deliverable"**, appears in **138 of 220 tasks**, always worth exactly 5 points. It is not a claim about the submission that a grader can mark true or false. It carries between 2.8% and 21.7% of a task's available score depending on which task you land on.
- **Four tasks can score arbitrarily negative.** The worst is a real estate comparative market analysis with a ceiling of +50 and penalty criteria totalling -380.
- **95 of 220 tasks (43%) ship with no reference files at all**, and 35 with no reference deliverable. Sixteen have neither.
- Three fields in the published rubric schema, `required`, `read_only`, and `form_content`, are null on all 10,453 criteria.
- Several things I expected to find wrong are not. Occupation coverage is exact, every criterion is human-authored, duplicate criteria are rare, and no task's prompt refers to an input file that is missing.

Everything above is exact and recomputable from the published parquet. The code is at the end.

## Anatomy of a task, and of a good rubric

The first task in the dataset is an audit engagement. The model is given a spreadsheet of 1,516 anti-financial-crime risk metrics and asked to select and document a statistical sample. Its rubric runs to 26 criteria.

That rubric is excellent, and it is worth being concrete about why, because it sets the standard the rest of the benchmark should be measured against. It names the sampling parameters: z = 1.645, p = 0.5, e = 0.10, with finite population correction. Those four numbers plus the population size fully determine the answer. I ran it: n₀ = 67.65, and after correction against N = 1,516 the required sample size is 65. There is exactly one right number and the rubric contains enough to derive it.

It goes further. Six criteria are conditional, of the form "if Cayman Islands occurs in the Country column, at least one such row is flagged as sampled." I checked all six against the actual spreadsheet. Marine Finance occurs 228 times, Correspondent Banking 28, Cayman Islands 21, Pakistan 34, UAE 59, and 401 rows have both quarters at zero. Five of the six guards fire on real data. The sixth names "UAE or United Arab Emirates" and only the abbreviation appears, which is not an error because the criterion accepts either.

This is a rubric written by somebody who did the work, opened the file, and checked their own conditions. Two careful graders reading the same submission against it will produce the same score.

Now the other end.

## 1. Points do not mean the same thing on different tasks

Criteria per task range from 14 to 137. Maximum attainable score ranges from 22 to 234, with a median of 72 and 95 distinct values across 220 tasks.

If GDPval scores are aggregated as raw points, a model that does well on the 234-point task is rewarded more than ten times as much as one that does equally well on the 22-point task, and the ranking becomes partly a fact about which tasks a model happens to suit. If they are normalized per task, then the granularity of the score varies by an order of magnitude: on a 14-criterion task the smallest possible increment is roughly 7% of the total, and on a 137-criterion task it is under 1%.

I want to be precise about what I can and cannot say here. The published dataset contains the rubrics; it does not contain the grading harness. Whether OpenAI normalizes per task before aggregating is not visible in what has been released, and I am not asserting they do not. What I am asserting is that the rubric layer as published carries no normalization, so anyone building on GDPval, and there are several such projects now, has to make that choice themselves, and the choice moves the number.

## 2. One line, 138 tasks, and no way to grade it

The single most reused rubric criterion in GDPval is this:

> Overall formatting and style of the deliverable

It appears 138 times, in 138 distinct tasks, which is 62.7% of the benchmark. It is always worth 5 points. It accounts for 690 points, 4.1% of every positive point in GDPval.

A rubric criterion has to be a proposition. It has to be a claim about the submission that is either true or false, so that two graders reading the same document reach the same verdict. "The workbook contains a worksheet named exactly 'Sample Size Calculation'" is a proposition. "Overall formatting and style of the deliverable" is a topic with a price tag. There is nothing to mark.

Its weight is not constant either, because it is a flat 5 points against ceilings that vary tenfold. On the highest-ceiling task it is 2.8% of the score. On the lowest it is 21.7%. So the amount of any given task's score that rests on an ungradeable line is itself an artifact of how long that task's rubric happens to be.

There is a charitable reading, which is that this line is a hook for a human grader's holistic impression and everyone involved knows it. I think that reading is probably right. It still means that on nearly two thirds of GDPval, a fixed slice of the score is grader impression rather than measurement, and that slice is larger on exactly the tasks where the rubric is thinnest.

## 3. Four tasks have no floor

Eighteen tasks (8.2%) contain penalty criteria, 94 in total. In most of them the penalties are modest and clearly intended to punish specific failures.

In four tasks, the penalties exceed the maximum attainable score. The extreme case is a Real Estate Brokers task asking for a comparative market analysis. Its rubric has 10 positive criteria worth 50 points and 38 penalty criteria worth -380. The penalties are structured per data category and per comparable property: "Includes an empty address for any sold comp", "Fails to identify the lot size for any sold comp", "Fails to identify the year built for any sold comp", each at -10.

These are not independent failures. A submission that produces the comps table but leaves it sparsely populated trips a dozen of them at once. A submission that omits the table entirely trips all 38. The floor is -380 against a ceiling of +50, and the distance between a mediocre answer and an empty one is more than seven times the distance between an empty answer and a perfect one.

What that does to an aggregate depends entirely on how the harness clamps, and again, the harness is not published. If negative task scores propagate into a mean, a single model failure on this one task moves the benchmark average more than success on any five other tasks.

## 4. Forty-three percent of tasks come with nothing attached

95 of the 220 tasks have zero reference files. 35 have no reference deliverable, meaning no professional-produced answer to compare against. Sixteen have neither.

This sharpens Epoch's February point rather than contradicting it. Their argument was that GDPval tasks sit outside a business context. The stronger version is that on 43% of them there is no context at all: the model is given a prompt and asked to invent the underlying material. For an occupation like Accountants and Auditors, the difference between "here is the population file, sample it" and "write a sampling memo" is the difference between doing the work and describing it.

The tasks that do carry material carry a lot of it, up to 17 files. The distribution is bimodal, and the benchmark's name suggests the loaded end is the part that matters.

## 5. Small things

Three fields in the rubric schema, `required`, `read_only` and `form_content`, are null on all 10,453 items. They are presumably live in OpenAI's internal grading tool and were stripped or never populated on release. A consumer of the public dataset cannot tell whether a criterion was mandatory.

Duplicate criteria within a single task occur 8 times across 7 tasks, which is 0.08% of criteria. That is a clean result and I report it because I went looking for it expecting worse.

Every one of the 10,453 criteria is marked `author_type: human`. There is no model-written rubric content in the public release.

## What this does and does not mean

It does not mean GDPval is broken, and it does not mean the reported numbers are wrong. Most of these rubrics are careful, the occupational coverage is exactly balanced at five tasks for each of 44 occupations, and the best of them are more rigorous than the working standards of the professions they model.

It means the instrument is not uniform, and the non-uniformity is invisible in a headline win rate. A GDPval score is an average over 220 tasks whose scoring scales differ by a factor of ten, where 63% of tasks reserve a fixed slice for grader impression, where four tasks have unbounded downside, and where 43% supply no input material. Those are not reasons to discard the benchmark. They are reasons a two-point difference between two models is not obviously a difference in capability, and reasons that anyone comparing GDPval scores across papers should check that both used the same aggregation.

The cheapest fixes are also the most obvious. Normalize per task, or publish the normalization. Replace the recurring formatting line with the two or three testable properties it stands in for, or drop its points. Clamp task scores at zero, or state that they are not clamped. None of that requires rewriting a single task.

## Method and limitations

Every figure in this review is computed from `openai/gdpval` on Hugging Face, revision `main`, retrieved 5 August 2026. The analysis is in `audit/structure.py` and takes about two seconds. `results/structure.json` holds the output.

The sampling verification in the second section downloads one reference workbook and recomputes the required sample size from the parameters stated in the rubric. That is the only claim here that touches file contents.

Three limitations matter.

The grading harness is not public, so I cannot see aggregation, clamping, or normalization. Every claim about what these rubric properties do to a final score is conditional on that, and I have tried to mark it each time.

I built a classifier to estimate what share of criteria are subjective rather than checkable, and it did not survive validation. Reading its output, it labelled plainly gradeable lines like "The Margin Impact pie chart contains exactly two slices labeled Cost and Investment" as unanchored merely because they contain no quoted string or numeral, and flagged "The report contains a clearly labeled Results section" as subjective on the strength of the word "clearly". Its numbers are not in this review. The code is in `audit/verifiability.py` with the failure documented, because a negative methodological result is still a result, and because the honest version of the subjectivity question needs hand-labelling that I have not done.

Finally, I have audited the rubrics, not the tasks. Whether GDPval's 220 prompts are representative of the 44 occupations, and whether the reference deliverables are actually good, are separate questions requiring occupational expertise I have only for a couple of these fields.

---

Code, data and results: [github.com/wolfvswhale/gdpval-audit](https://github.com/wolfvswhale/gdpval-audit)
Prior work: [Epoch AI, "What do 'economic value' benchmarks tell us?"](https://epoch.ai/blog/what-do-economic-value-benchmarks-tell-us), February 2026


## Changelog

**Version 1.1, 20 August 2026.** Re-checked against the source. The openai/gdpval dataset on Hugging Face reports a last-modified date of 10 February 2026, earlier than the 5 August 2026 retrieval this review was computed from, so every figure above still recomputes unchanged from the same revision.

**Version 1.0, 6 August 2026.** First published.
