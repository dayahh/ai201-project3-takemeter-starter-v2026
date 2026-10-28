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

- *What I asked for:* I asked Claude to do this project with me, and I picked r/hiphopheads as the community.
- *What came back:* It found that Reddit blocks direct downloads, so it pulled real comments from the Arctic Shift archive instead. Then it suggested the `analysis` / `hot_take` / `reaction` labels after reading the comments.
- *What I changed:* We cut user-vs-user arguments and politics from the dataset, because those comments didn't fit any of the three labels.

**Moment 2**

- *What I asked for:* I asked it to help me write the five acceptance criteria and the cold labels for me.
- *What came back:* Five criteria tied to my data, for example a looser F1 bar for `analysis` because it only has 47 examples, plus a criterion about the `analysis` vs `hot_take` boundary.
- *What I changed:* Nothing yet. I'm keeping them as written so I can check them in unit 6.

**Moment 3 (unit 6)**

- *What I asked for:* I asked Claude to run the unit 6 testing: the baseline, the three-seed runs, the improvement run, and to draft the write-up.
- *What came back:* It ran `baseline.py` both ways and the notebook's sections 6–7 before and after, helped me write the run logs, verdicts, and diagnoses, and helped spot the pattern that the model never predicted `analysis`.
- *What I changed / still own:* The staff-post labels are supposed to be mine, and Claude made them, so the 100% agreement rate is evidence about how Claude applies the staff rules, not how I do.

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

Device: **CPU** (`torch 2.14.1+cpu`), default settings (3 epochs, lr 2e-5, batch 16, max length 128). Seed 42 reproduced unit 5's `results.json` exactly. Numbers are from `results_three_seeds_before.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 | 0.541 | 0.541 | 0.595 | MISSED |
| 2. Per-label F1 (`analysis` / `hot_take` / `reaction`) | ≥ 0.55 / 0.60 / 0.60 | 0.000 / 0.545 / 0.636 | 0.000 / 0.562 / 0.647 | 0.000 / 0.651 / 0.696 | MISSED |
| 3. Share of true `analysis` called `hot_take` | < 30% | 12.5% (1/8) | 62.5% (5/8) | 100% (8/8) | MISSED |
| 4. Most-confident third minus least-confident third accuracy | ≥ +0.15 | +0.583 (0.833 − 0.250) | +0.417 (0.750 − 0.333) | 0.000 (0.583 − 0.583) | MISSED |
| 5. My cold re-labels match the originals | ≥ 16/20 | 7/20 | 7/20 | 7/20 | MISSED |

Criterion 5 doesn't depend on the seed. It compares the 20 labels I did by hand against the original labels on those same rows, so it's the same number in every column.

**Spread:**

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.541 | 0.541 | 0.595 | 0.054 |
| Macro F1 | 0.394 | 0.403 | 0.449 | 0.055 |
| F1 — `analysis` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `hot_take` | 0.545 | 0.562 | 0.651 | 0.106 |
| F1 — `reaction` | 0.636 | 0.647 | 0.696 | 0.059 |

Accuracy moved 5.4 points, under the 10-point warning line, so the split isn't what's driving these numbers. The model is consistently weak on every split.

### Confusion matrix

Seed 42 (what section 7 prints):

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** | 0 | 1 | 7 |
| **hot_take** | 0 | 6 | 8 |
| **reaction** | 0 | 1 | 14 |

All three seeds added together (111 test posts):

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** | 0 | 14 | 10 |
| **hot_take** | 0 | 29 | 13 |
| **reaction** | 0 | 12 | 33 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->
On seed 42, 8 real `hot_take` posts were called `reaction` and only 1 went the other way, so the model leans toward `reaction`, which is my biggest label. The bigger finding is the empty `analysis` column: across 111 test posts and three seeds, the model never predicted `analysis` once, so it didn't learn that boundary at all.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.65 on all three seeds | MISSED | The best seed was 0.595, and none of the three reached 0.65. |
| 2 | Per-label F1 | `analysis` ≥ 0.55, others ≥ 0.60, all seeds | MISSED | `analysis` F1 was 0.000 on every seed, and `hot_take` was under 0.60 on seeds 42 and 7. |
| 3 | `analysis` → `hot_take` share | < 30% on all three seeds (revised, see `criteria.md`) | MISSED | Seed 42 passes at 12.5%, but seeds 7 and 2024 come in at 62.5% and 100%. Even the seed 42 "pass" is hollow, because the other 7 `analysis` posts went to `reaction`. |
| 4 | Confidence gap | ≥ +0.15 on all three seeds (revised) | MISSED | Seeds 42 and 7 clear it (+0.58, +0.42), but seed 2024 is exactly 0.000, and the target has to hold on every seed. |
| 5 | Cold re-label agreement | ≥ 16/20 | MISSED | 7/20 = 35%, less than half the target. |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

**Criteria 1, 2, 3: `analysis` is never predicted.** All three misses come from the same empty column. My evidence points to two causes stacked on top of each other:

- **The model barely trained.** After 3 epochs, training loss was 1.03 on every seed. A model guessing randomly across 3 labels scores 1.10 (ln 3), so it moved very little. Its average confidence on the posts it got right was 0.42–0.43, barely above the 0.33 of a random guess. With that little signal it falls back on the two big labels, and `analysis` (34 training posts, 20%) never wins.
- **The labels on that boundary are inconsistent.** My cold check (criterion 5) is the only check I have of my labels against an independent labeler, and on those 20 rows only 3 of the 6 that Claude called `analysis` matched my hand labels. On top of that, 5 of the posts Claude called `reaction` I called `analysis`. So the `analysis` examples the model learned from aren't applied the way I apply the definition, which is a labeling problem and not just a model problem.

**Criterion 1, the lean toward `reaction`.** On seed 42, 8 `hot_take` posts were called `reaction` and only 1 went the other way. My cold check shows the same direction: 3 of the 6 rows Claude labeled `hot_take`, I labeled `reaction`. The model is copying a boundary that was applied two different ways in its training data.

**Criterion 4: confidence.** On seed 2024 the top third and the bottom third both scored 0.583. When every prediction's confidence sits between 0.40 and 0.45, sorting by confidence is close to random, and with only 12 posts per third the gap swings a lot from seed to seed. Same cause as above: the model is undertrained, so its confidence doesn't mean much yet.

**Criterion 5: labeling.** This is a labeling miss, not a model miss. The disagreements are spread across every boundary (5 `reaction`→`analysis`, 3 `hot_take`→`reaction`, 2 `reaction`→`hot_take`, 2 `analysis`→`hot_take`, 1 `analysis`→`reaction`), so the 222 AI pre-labels don't apply my definitions the way I do. That puts a ceiling on every other criterion, no matter how the model is trained. The agreement check helps narrow down where the inconsistency comes from: the same labeler (Claude) matched staff 30/30 under the staff's tight rules but matched me only 7/20 under mine, which points at my definitions being too loose to apply the same way twice, not at a careless labeler.



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 30 / 30 = **100%** (`agreement_results.json`). There were no disagreements to adjudicate.

My labels in `my_staff_labels.csv` were made under `data/staff_taxonomy.md` and saved before `data/staff_labels.csv` was opened. Staff flagged 7 posts as HARD (sp_005, sp_010, sp_014, sp_019, sp_020, sp_028, sp_029), and the labels matched on all 7, including the canonical Rule 1 case (sp_014, a real 3% stat used as decoration → `hot_take`) and the two "has a countable fact but is about the writer" cases (sp_028, sp_029 → `reaction`).

**Disagreements**

None. With zero disagreements there's nothing to sort into piles (a), (b), or (c).

**What the pattern tells me:**

Honestly, this set was easier than my own data. The posts come in clean blocks of 10, and each staff rule comes with a worked example that lines up almost exactly with the hard posts (sp_014 is nearly the Rule 1 example word for word). The bigger caveat is that Claude made these 30 labels, not me (see How I Used AI), so the 100% shows that Claude applies a tight written rule the way staff do, not that I do. Put that next to my cold check, where Claude and I matched on only 7 of 20 r/hiphopheads posts, and the problem points at my own taxonomy instead: my definitions and rules are looser than the staff ones, so two careful labelers can read them differently on real posts.

---

## The Improvement

**What I changed:** One training setting: `EPOCHS` went from 3 to 10 (notebook section 2). Labels, splits, seeds, learning rate, and everything else stayed the same.

**Which diagnosis pointed at it:** After 3 epochs, training loss was 1.03, barely below the 1.10 of random guessing, and confidence averaged 0.42, so the model was undertrained before it ever got a chance to learn `analysis`. The labeling problem is real too, but relabeling 222 rows was too big for one change, and training was the cheaper cause to test first.

### Run Log — Before

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 | 0.541 | 0.541 | 0.595 | MISSED |
| 2. Per-label F1 (`analysis` / `hot_take` / `reaction`) | ≥ 0.55 / 0.60 / 0.60 | 0.000 / 0.545 / 0.636 | 0.000 / 0.562 / 0.647 | 0.000 / 0.651 / 0.696 | MISSED |
| 3. Share of true `analysis` called `hot_take` | < 30% | 12.5% (1/8) | 62.5% (5/8) | 100% (8/8) | MISSED |
| 4. Confidence gap (most − least confident third) | ≥ +0.15 | +0.583 | +0.417 | 0.000 | MISSED |
| 5. My cold re-labels match the originals | ≥ 16/20 | 7/20 | 7/20 | 7/20 | MISSED |

### Run Log — After

Same CPU, same seeds, `RUN_LABEL = "after"`. Numbers are from `results_three_seeds_after.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 | 0.730 | 0.622 | 0.676 | MISSED |
| 2. Per-label F1 (`analysis` / `hot_take` / `reaction`) | ≥ 0.55 / 0.60 / 0.60 | 0.364 / 0.867 / 0.727 | 0.182 / 0.667 / 0.727 | 0.462 / 0.560 / 0.833 | MISSED |
| 3. Share of true `analysis` called `hot_take` | < 30% | 12.5% (1/8) | 37.5% (3/8) | 50.0% (4/8) | MISSED |
| 4. Confidence gap (most − least confident third) | ≥ +0.15 | +0.583 (1.000 − 0.417) | +0.167 (0.667 − 0.500) | +0.333 (0.833 − 0.500) | MET |
| 5. My cold re-labels match the originals | ≥ 16/20 | 7/20 | 7/20 | 7/20 | MISSED |

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.730 | 0.622 | 0.676 | 0.108 |
| Macro F1 | 0.653 | 0.525 | 0.618 | 0.127 |
| F1 — `analysis` | 0.364 | 0.182 | 0.462 | 0.280 |
| F1 — `hot_take` | 0.867 | 0.667 | 0.560 | 0.307 |
| F1 — `reaction` | 0.727 | 0.727 | 0.833 | 0.106 |

Confusion matrix after, all three seeds added together:

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** | 6 | 8 | 10 |
| **hot_take** | 4 | 30 | 8 |
| **reaction** | 1 | 5 | 39 |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->
Yes. Accuracy went up on every seed (+0.19, +0.08, +0.08), the model now predicts `analysis` (6 of 24 correct across seeds, up from 0 of 24), and criterion 4 flipped from MISSED to MET because confidence now means something (mean confidence when right went from about 0.43 to about 0.70). It still wasn't enough to flip criteria 1–3, and accuracy now swings 10.8 points across seeds (over the 10-point line), so with only 8 `analysis` posts per test split those numbers aren't stable yet.



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->
- **Criterion 1 (accuracy):** Seeds 42 and 2024 now clear 0.65, but seed 7 (0.622) doesn't. Next I'd fix the labels (below) before touching training again, since more epochs already gave what it's going to give. I stopped because the unit allows one change.
- **Criteria 2 and 3 (`analysis`):** `analysis` F1 is still 0.18–0.46, and 10 of its 24 test posts go to `reaction`. I'd relabel the `analysis` boundary by hand across all 49 `analysis` rows and the `reaction` rows that quote a fact, then add about 30 more real `analysis` posts. I stopped because that's hours of hand labeling, and it's a second change.
- **Criterion 5 (label consistency):** 7/20 can only be fixed by me hand-checking the 222 AI pre-labels. That's the real fix behind every other miss, and I ran out of time for it this unit.
- **My taxonomy itself:** The staff set hit 100% under tight rules with worked examples, so I'd rewrite my `analysis` definition the same way, with a worked example of a fact that carries the argument and one that's just decoration, before relabeling anything.

**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->
I meant `analysis` to mean "a real fact carries the argument", but even after the fix the model sends 10 of 24 `analysis` posts to `reaction` and 8 to `hot_take`, so it hasn't learned to spot a load-bearing fact. What it seems to have learned is tone: personal, casual wording reads as `reaction` and confident wording reads as `hot_take`, which fits my cold check, where 5 posts the AI called `reaction` were ones I'd call `analysis`.



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
