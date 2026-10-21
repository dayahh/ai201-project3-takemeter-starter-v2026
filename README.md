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
> **The rest of this file is your submission.** Fill it in as you go.

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

TakeMeter sorts comments from **r/hiphopheads**, a big public subreddit about hip-hop. It puts each comment into one of three labels: `analysis`, `hot_take`, or `reaction`. The goal is to separate comments that back up a point with a real fact from comments that just state an opinion, or that are just someone sharing how they feel. While reading the sub I noticed three things: some comments cite numbers like streams, sales, dates or credits, most just declare something is trash or the GOAT, and a lot are people talking about their own memories of an album.

---

## Label Taxonomy

### `analysis`

**Definition:** Makes a point about music, artists, or the scene and backs it up with a specific fact you could check, like a number, a date, a credit, a setlist, or a quoted lyric.

**Example 1:**
> As of today he has 71.1 million monthly listeners on Spotify alone, his new song off of bully “father” is number 1 globally on Apple Music, bully is the number 1 album on both Spotify and Apple Music, and he’s projected to sell 250k-300k first week. You are delusional if you think anyone actually stopped listening

**Example 2:**
> Try reading ... it says "in 1996" lol... Pusha Twas like 19 years old and hadn't put out any music yet . Reasonable Doubt came out in June 1996.. no one even knew Jay z at that time

### `hot_take`

**Definition:** A confident general claim about music, artists, or the scene with no checkable support. It might be true, but it isn't argued.

**Example 1:**
> Beat is fucking immaculate, but Travis cemented himself as one of the worst rappers out with this one.

**Example 2:**
> BSBD is the most talented production group in rap history imo, I still don't think anything else has ever reached the production level that Noir did.

### `reaction`

**Definition:** The writer's own personal response, like a feeling, a listening memory, a favorites list, or a first impression, not a claim about how the world is.

**Example 1:**
> This set gave me goosebumps! He crammed so many hits into an hour, his voice sounded amazing, his dancing was euphoric and the band were incredible. It's sad that his tour got cancelled, but in a way this is the perfect send off for the Gambino name (for now).

**Example 2:**
> Ngl I liked only 2 songs from the album, shopping and flamethrower the rest sound kinda alienish

### The hardest boundary

**Which two labels:** `analysis` vs `hot_take`

**The decision rule I used every time:** If the post has a checkable fact AND that fact is the reason for the claim, it's `analysis`. If the fact is just decoration on an opinion the writer already had, or the "number" is made up (like "a rapper can have 2 million listeners and nobody knows them"), it's `hot_take`. For `hot_take` vs `reaction`, I ask if the post is about the world (`hot_take`) or about the writer's own experience (`reaction`), and if it's both I go with whichever one most of the words are spent on.

The borderline post that made me write this rule:
> We lost both Juice and XXX at 21 and both had potential to take over the hip hop world.

The age is a real fact, but it doesn't prove the "potential to take over" part, so it's `hot_take`.

---

## The Dataset

**Where the posts came from:** Public comments from r/hiphopheads, posted between October 2025 and September 2026. I pulled them from the Arctic Shift Reddit archive and kept whole comments between 80 and 600 characters, with no links. From a pool of about 730, I kept the ones about music, artists, or the rap scene, and skipped user-vs-user arguments, politics, and off-topic stuff like video games, since none of my labels fit those.

**How I labelled them:** The first 20 rows in `labels.csv` are marked `cold`. Every other row was pre-labeled by an AI (Claude) using my definitions and then checked one by one, and is marked `pre-labeled by Claude, checked` in the note column. See the disclosure under How I Used AI, because the cold rows were not done fully by hand.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `analysis` | 47 | 19.4% |
| `hot_take` | 91 | 37.6% |
| `reaction` | 104 | 43.0% |
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

**Anything I changed from the defaults, and why:** Nothing. I kept all the default settings so this first run is a clean starting point to compare against in unit 6.

**Split sizes:** train 168 / val 37 / test 37.

| Label | train | val | test |
|---|---|---|---|
| `analysis` | 33 | 7 | 7 |
| `hot_take` | 63 | 14 | 14 |
| `reaction` | 72 | 16 | 16 |

`analysis` has only 7 posts in the test split, which is under 8, so its F1 will probably jump around a lot between seeds.

---

## How I Used AI

**Moment 1**

- *What I asked for:* I asked Claude (in Claude Code) to do this project with me, and I picked r/hiphopheads as the community.
- *What came back:* It found that Reddit blocks direct downloads, so it pulled real comments from the Arctic Shift archive instead. Then it suggested the `analysis` / `hot_take` / `reaction` labels after reading the comments.
- *What I changed:* We cut user-vs-user arguments and politics from the dataset, because those comments didn't fit any of the three labels.

**Moment 2**

- *What I asked for:* I asked it to write the five acceptance criteria and the cold labels for me, since this was practice.
- *What came back:* Five criteria tied to my data, for example a looser F1 bar for `analysis` because it only has 47 examples, plus a criterion about the `analysis` vs `hot_take` boundary.
- *What I changed:* Nothing yet. I'm keeping them as written so I can check them honestly in unit 6.

**Pre-labelling disclosure:** All 242 labels in `labels.csv` were made by Claude using the definitions above, including the 20 rows marked `cold`. Those 20 were labeled first, before any other rows, but by the AI and not by me by hand. The criteria in `criteria.md` were also written by Claude. To make the unit 6 agreement check mean something, I plan to re-label the 20 cold rows myself without looking.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



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
