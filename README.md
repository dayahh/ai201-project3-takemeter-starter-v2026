# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

TakeMeter sorts comments from **r/hiphopheads**, a big public subreddit about hip-hop. It puts each comment into one of three labels: `analysis`, `hot_take`, or `reaction`. The goal is to separate comments that back up a point with a real fact from comments that just state an opinion, or that are just someone sharing how they feel.

---

## Label Taxonomy

### `analysis`

**Definition:** Makes a point about music, artists, or the scene and backs it up with a specific fact you could check, like a number, a date, a credit, a setlist, or a quoted lyric.

**Example 1:**
> Goldlink bit a whole segment of a verse on "Meditation" from an unreleased Smino track and got called out for it  In hindsight it seems blatant, the beige loaf line definitely screams smino and is too clever to be a goldlink bar

**Example 2:**
> Try reading ... it says "in 1996" lol... Pusha Twas like 19 years old and hadn't put out any music yet . Reasonable Doubt came out in June 1996.. no one even knew Jay z at that time

### `hot_take`

**Definition:** A confident general claim about music, artists, or the scene with no checkable support. It might be true, but it isn't argued.

**Example 1:**
> Please stop comparing Drake to Kanye I don’t know why we’re still doing this 4+ years later, they’re different animals and Kanye washes

**Example 2:**
> BSBD is the most talented production group in rap history imo, I still don't think anything else has ever reached the production level that Noir did.

### `reaction`

**Definition:** The writer's own personal response, like a feeling, a listening memory, a favorites list, or a first impression, not a claim about how the world is.

**Example 1:**
> TLOP was a time, one of the albums in my life I vividly remember it dropping along with astroworld

**Example 2:**
> there’s been so many times this song has broken me into pieces, but there’s so many times it put me back together.

### The hardest boundary

**Which two labels:** `analysis` vs `hot_take`

**The decision rule I used every time:** If the post has a checkable fact AND that fact is the reason for the claim, it's `analysis`. If the fact is just decoration on an opinion the writer already had, or the "number" is made up (like "a rapper can have 2 million listeners and nobody knows them"), it's `hot_take`. For `hot_take` vs `reaction`, I ask if the post is about the world (`hot_take`) or about the writer's own experience (`reaction`), and if it's both I go with whichever one most of the words are spent on.

The borderline post that made me write this rule:
> We lost both Juice and XXX at 21 and both had potential to take over the hip hop world.

The age is a real fact, but it doesn't prove the "potential to take over" part, so it's `hot_take`.

---

## The Dataset

**Where the posts came from:** Public comments from r/hiphopheads, posted between October 2025 and September 2026. I pulled them from the Arctic Shift Reddit archive and kept whole comments between 80 and 600 characters, with no links. From a pool of about 730, I kept the ones about music, artists, or the rap scene, and skipped user-vs-user arguments, politics, and off-topic stuff like video games, since none of my labels fit those.

**How I labelled them:** The first 20 rows in `labels.csv` are marked `cold`, and I labeled those by hand with no AI help. Every other row was pre-labeled by an AI (Claude) using my definitions, and is marked `pre-labeled by Claude, checked` in the note column. See the disclosure under How I Used AI.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `analysis` | 49 | 20.2% |
| `hot_take` | 92 | 38.0% |
| `reaction` | 101 | 41.7% |
| **Total** | 242 | 100% |

**Three hard cases**

**1.**
> *The post:* "1.4 billion people in China, now that restrictions are being lifted youre going to see every major artist go there"
>
> *Could have been:* `analysis` or `hot_take`
>
> *I chose, because:* `analysis`. The 1.4 billion number is the actual reason given for the prediction, so the fact carries the argument instead of just decorating it.

**2.**
> *The post:* "the definition just shifted i think, underground used to mean you literally cannot find it anywhere unless you dig for it in the crates or on blogs. now it just mean you aint in the mainstream conversation even if you pulling numbers. like a rapper can have 2 million monthly listeners and still nobody at the barbershop know who they are"
>
> *Could have been:* `analysis` or `hot_take`
>
> *I chose, because:* `hot_take`. It has a number, but "2 million monthly listeners" is a made-up example, not a stat about a real artist, so there's nothing to check.

**3.**
> *The post:* "I genuinely like the album. I’ve listened to it maybe 50 times now. My only qualm is that some tracks are a little too polished and bubblegum pop like. ..." (Isaiah Rashad album thread)
>
> *Could have been:* `reaction` or `hot_take`
>
> *I chose, because:* `reaction`. It does make claims about the songs being too poppy, but most of the post is about the writer's own listening and taste, so I used the "most words" tie-break.

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128 tokens, seed 42. Trained on **CPU** (`device: cpu` in `results.json`), and it took about 80 seconds.

**Anything I changed from the defaults, and why:** Nothing. I kept all the default settings so this first run is a clean starting point to compare against in unit 6. I trained twice with the same settings: once with the AI's labels on the 20 cold rows, then again after I swapped in my own hand labels. `results.json` is from the second run.

**Split sizes:** train 168 / val 37 / test 37.

| Label | train | val | test |
|---|---|---|---|
| `analysis` | 34 | 7 | 8 |
| `hot_take` | 64 | 14 | 14 |
| `reaction` | 70 | 16 | 15 |

`analysis` has only 8 posts in the test split, which is right at the edge, so its F1 will probably jump around a lot between seeds.

---

## How I Used AI

**Moment 1**

- *What I asked for:* I asked Claude (in Claude Code) to do this project with me, and I picked r/hiphopheads as the community.
- *What came back:* It found that Reddit blocks direct downloads, so it pulled real comments from the Arctic Shift archive instead. Then it suggested the `analysis` / `hot_take` / `reaction` labels after reading the comments.
- *What I changed:* We cut user-vs-user arguments and politics from the dataset, because those comments didn't fit any of the three labels.

**Moment 2**

- *What I asked for:* I asked it to help me write the five acceptance criteria and the cold labels for me.
- *What came back:* Five criteria tied to my data, for example a looser F1 bar for `analysis` because it only has 47 examples, plus a criterion about the `analysis` vs `hot_take` boundary.
- *What I changed:* Nothing yet. I'm keeping them as written so I can check them in unit 6.

**Pre-labelling disclosure:** 222 of the 242 labels in `labels.csv` were made by Claude using the definitions above. The criteria in `criteria.md` were also written by Claude. Claude first labeled the 20 `cold` rows too, but then I re-labeled them myself by hand without looking at its answers (mine are in `my_cold_labels.csv`), and `labels.csv` now uses my labels for those 20.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

Baseline is `facebook/bart-large-mnli` zero-shot with my definitions from `label_definitions.txt`, scored on the same 37 posts in `test_split.csv`. Trained is `results.json` (seed 42).

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.243 | 0.541 | +0.297 |
| Macro F1 | 0.166 | 0.394 | +0.228 |
| F1 — `analysis` | 0.364 | 0.000 | -0.364 |
| F1 — `hot_take` | 0.133 | 0.545 | +0.412 |
| F1 — `reaction` | 0.000 | 0.636 | +0.636 |

**Definitions vs. bare label names:** I ran the baseline a second time without `label_definitions.txt`. With bare names it got **0.297** accuracy (F1 `analysis` 0.000, `hot_take` 0.378, `reaction` 0.276), so my definitions were worth **−0.054**, meaning they made it worse. The table above is the definitions run.

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->
I predict my trained model beats the baseline on `reaction`, because it saw 70 examples of how people on r/hiphopheads talk about their own listening, but the baseline beats it on `analysis`, because my trained model never predicted `analysis` once in unit 5 and the baseline at least gets the word "fact" in its prompt.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->
My prediction was right in both directions. The baseline called 36 of 37 posts `analysis`, because almost every post "makes a point about music", so its 0.364 F1 on `analysis` comes from guessing it every time, not from actually spotting facts. Fine-tuning more than doubled accuracy (+0.30), so training clearly added something, but the trained model's own `analysis` F1 is 0. That means neither model has learned the one boundary this project is built around.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
