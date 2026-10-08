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

| Area                  | A question it could answer                                            |
| --------------------- | --------------------------------------------------------------------- |
| Overall accuracy      | How often does it need to be right to be worth using?                 |
| Per-label performance | Is one label allowed to be much worse than the others?                |
| Balance               | How lopsided can your label counts get before it's a problem?         |
| Consistency           | If someone else labelled the same posts, how often should you agree?  |
| Confidence            | Should a confident prediction be right more often than an unsure one? |

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

## 1. Overall accuracy

Overall accuracy of at least **0.80** on the held-out test split, across all three seeds.

**Why this target:** My submission-only data currently sits at `reaction` 85.2% (121/142).
Even after pulling comment-thread data to get every label under the 70% dominance cap,
`reaction` will likely still be my biggest label by a wide margin — which means a model that
just always guesses `reaction` gets roughly 70% accuracy "for free," without learning
anything. A target of 0.80 forces the model to actually use `analysis` and `hot_take`
signal, not just ride the majority class.

---

## 2. Per-label performance

Every label has an F1 of at least **0.50** on the held-out test split, across all three seeds.

**Why this target:** `hot_take` is my smallest label right now (6 of 142 posts, 4.2%), and
even after rebalancing with comment threads it'll likely stay my smallest. At ~200 total
posts with a 70/15/15 split, that could mean single digits of `hot_take` in the test fold —
an F1 computed on that few examples swings hard between seeds. 0.50 is clearly above
chance-level for a 3-way problem but loose enough to survive that jumpiness; I'm not holding
`hot_take` to the same bar as `reaction`.

---

## 3. Balance

No label exceeds **70%** of the final ~200-post dataset, and the smallest label has at least
**30** examples before the split.

**Why this target:** The 70% figure is the dominance cap `check_dataset` already enforces,
so this is really about the second number. 30 examples in the smallest label means roughly
4-5 of them survive into the 15% test fold — few enough to be noisy (see criterion 2), but
not so few that the label's test score is meaningless. My first pass (submissions only) had
`hot_take` at just 6 examples total, which is why pulling comment threads for `analysis` and
`hot_take` specifically is the plan, not just adding more random submissions.

---

## 4. Consistency

Agree with the staff labels on at least **75%** of the 30-post staff set (`agreement.py`).

**Why this target:** Writing the `reaction` vs. `hot_take` decision rule (verdict vs. feeling,
in `sample_posts.md`) only became necessary because that boundary wasn't obvious on first
read — Roy Keane's "it was cheating" quote could plausibly go either way depending on whether
you weight timing or content. I expect staff to hit similar genuine boundary cases, so 75%
leaves room for a handful of defensible disagreements without treating normal boundary
friction as a failure.

---

## 5. Confidence

Among predictions where the model's top-class probability is at least 0.90, accuracy should be at least **0.85** — meaningfully higher than overall accuracy.

**Why this target:** If high-confidence predictions aren't noticeably more accurate than the
overall rate, the confidence score isn't telling me anything real, and I can't trust it to
flag which predictions need a human to double check. 0.85 against an overall-accuracy target
of 0.80 (criterion 1) is a deliberately modest gap — enough to prove confidence is
informative, not enough to assume the model is well-calibrated on 200 examples.

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
