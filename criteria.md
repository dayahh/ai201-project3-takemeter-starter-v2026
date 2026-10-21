# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy is at least 0.65

On the held-out test split, overall accuracy is at least **0.65** on every one of the three seeds.

**Why this target:** My biggest label is `reaction` at about 43% of the data, so a model that always guesses `reaction` already gets around 0.43. I want the model to clearly beat that, and 0.65 is the point where I'd actually trust it to sort a page of comments for me.

---

## 2. Every label has an F1 of at least 0.55, and `hot_take` / `reaction` reach 0.60

On the held-out test split, `analysis` has an F1 of at least **0.55**, and `hot_take` and `reaction` each have an F1 of at least **0.60**, on all three seeds.

**Why this target:** `analysis` is my smallest label with only 47 examples, so only about 7 land in the test split and its score will jump around a lot. I gave it a slightly looser bar than the two bigger labels, which have over 90 examples each.

---

## 3. Fewer than 30% of real `analysis` posts get called `hot_take`

In the confusion matrix for seed 42, fewer than **30%** of the true `analysis` posts in the test split are predicted as `hot_take`.

**Why this target:** `analysis` vs `hot_take` is my hardest boundary, since both are confident claims and the only difference is whether a real fact carries the argument. If the model mixes those two up a lot, it didn't learn the one thing this classifier is for.

---

## 4. Confident predictions are right more often than unsure ones

In `results.json`, `accuracy_most_confident_third` is at least **0.15 higher** than `accuracy_least_confident_third`.

**Why this target:** If the model's confidence means anything, the posts it's surest about should be right a lot more often. A gap of 0.15 is big enough that it isn't just noise on a test split of about 37 posts (12 per third).

---

## 5. I agree with my own cold labels at least 80% of the time

When I re-label the 20 rows marked `cold` in `labels.csv` in unit 6, without looking at the original labels, at least **16 of 20 (80%)** match the original.

**Why this target:** If I can't apply my own definitions the same way twice, the model is learning from noisy labels and none of the other numbers mean much. 80% leaves room for the few posts that really sit on the `hot_take` / `reaction` line.

---


<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
