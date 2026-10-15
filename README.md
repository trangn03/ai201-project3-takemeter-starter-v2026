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

This project labels posts from r/soccer, a public subreddit that needs no
login to read. Most posts there are either a reported event with no
argument attached, a quote reacting to something that just happened, or a
fan making an actual case for a position. The classifier sorts each post
into one of three labels — `analysis`, `hot_take`, or `reaction` — based on
whether it states a verdict, and whether that verdict is backed by a
specific, checkable fact.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** States a claim where a specific, checkable fact (a stat, a
date, a quote) is the reason given for the claim — remove the fact and the
claim stops holding up.

**Example 1:**

> Jamie Carragher: "All of Manchester City's titles, they've won in this
> time (of cheating) or trophies should be taken off their list... I
> remember the referee blowing a final whistle of a big game. I remember
> Jerzy Dudek saving the penalty, that feeling I got. Those City players
> will never get that feeling. And it was ROBBED by Manchester City."

**Example 2:**

> 10 years in the PL without a single title is a poor return for a manager
> of his ability

### `hot_take`

**Definition:** States or clearly implies a confident verdict (including a
rhetorical question with an obvious implied answer) without a fact offered
as the reason for it. A fact can be present, but only decoratively — the
verdict doesn't depend on it.

**Example 1:**

> Roy Keane on Man City verdict: "If I was a player, I'd be 100% sticking
> those medals in the bin. It's cheating. You hear nonsense from ex-players
> saying it was won on the pitch. It was cheating. It was cheating going
> on."

**Example 2:**

> Pep Guardiola's Man City was built on a false premise, funded by financial
> chicanery, for his benefit. That's why his legacy is completely tarnished

### `reaction`

**Definition:** Describes a scene, feeling, or event with no verdict stated
or implied. If you can't point to a sentence that asserts "X is
true/should happen," it's `reaction`, even if a number appears in the text.

**Example 1:**

> Tifos from Konyaspor fans in their friendly match against the Palestinian
> National Team.

**Example 2:**

> Lionel Scaloni in tears as the Argentine national anthem is played

### The hardest boundary

**Which two labels:** `reaction` and `hot_take`

**The decision rule I used every time:** If a quote renders a verdict — a
claim about what IS the case — it's `hot_take`, even if it was said in the
same breath as breaking news. `reaction` is reserved for quotes that stay at
the level of feeling (shock, relief, refusal to engage) without resolving
into a stated conclusion. Timing triggers the quote; content decides the
label. (Worked out on the Roy Keane quote above: it's timed exactly like a
`reaction` — said the moment the verdict dropped — but its content is a flat
verdict with no feeling-language, which is what `hot_take` is for.)

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** r/soccer (public, no login required), pulled via Reddit's
`.json` listing/comment endpoints.

**How I labelled them:** Go through the list and start the label, then split into two tracks. I
gave Claude `label_definitions.txt` and the collected posts, and had it
assign all 189 non-held-out rows to `analysis` / `hot_take` / `reaction`;
those rows are marked `pre-labelled, needs review` in the `note` column and
are being read and corrected individually, not accepted as-is. Separately, I
picked 20 posts at even intervals through the file and labelled those myself
cold — no AI, no tools, definitions only — so I'd have a set that reflects
only my own judgment for the agreement check next unit; those rows are
marked `cold` once I hand in my labels.

**Counts per label:**

| Label           | Count | Share |
| --------------- | ----- | ----- |
| `reaction`    | 134   | 64.1% |
| `hot_take`    | 41    | 19.6% |
| `analysis`    | 34    | 16.3% |
| **Total** | 209   | 100%  |

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**

> *The post:* Roy Keane on Man City's verdict: "If I was a player, I'd be
> 100% sticking those medals in the bin... It was cheating. It was cheating
> going on."
>
> *Could have been:* `reaction` or `hot_take` — it's a quote given the
> moment the verdict was announced, which is exactly what `reaction`
> describes.
>
> *I chose, because:* `hot_take` — the content isn't a feeling, it's a flat
> verdict stated twice with no fact attached. This is what produced the
> decision rule above: timing triggers the quote, content decides the
> label.

**2.**

> *The post:* "There's no way he could've known, he was only directly
> involved for a decade"
>
> *Could have been:* `hot_take` (reads like a one-line zinger) or `analysis`
> (the sarcasm's entire bite depends on the specific fact — "a decade" —
> remove it and the line doesn't land).
>
> *I chose, because:* `analysis`, but this is the one I'm least sure of —
> sarcasm built on a fact doesn't feel like the same thing as an argued
> claim, even though it passes my own "remove the fact" test.

**3.**

> *The post:* "He even managed to get them relegated. GOAT my ass!"
>
> *Could have been:* `hot_take` (read in isolation it's just an insult) or
> `analysis` (the "relegated" fact is the stated reason the "GOAT" claim is
> being rejected).
>
> *I chose, because:* `analysis` — but what made this hard wasn't the
> label, it was the post: the comment was actually about a different
> manager (Maresca) than the one the thread was about (Pep), which I only
> caught by re-reading the surrounding replies.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** seed 42, 3 epochs, lr 2e-5, batch size 16, max length 128

**Anything I changed from the defaults, and why:** Nothing — ran with the
notebook's defaults.

**Device:** CPU (`torch 2.14.1+cpu`, no CUDA/MPS available on this machine —
recorded in `results.json`'s `device`/`device_name` fields).

**Split sizes:** 209 total → 32 in the test split: `reaction` 21, `hot_take`
6, `analysis` 5. Both `hot_take` and `analysis` are under the ~8-example
floor for a stable test-split F1 — exactly the jumpiness criteria.md's
per-label target (criterion 2) was written to expect.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* I asked Claude to act as an outside stress-test: write
  eight synthetic r/soccer-style posts designed to sit on the boundaries of
  my three labels, given only `label_definitions.txt`, then try to classify
  them.
- *What came back:* 3 of the 8 posts wouldn't classify cleanly. The failure
  was always the same shape: a post can contain a checkable fact without
  that fact actually supporting the claim next to it (a stat sitting
  decoratively beside an unrelated verdict), and `analysis`'s original
  wording — "backed by a specific, checkable fact" — didn't distinguish
  that from a fact that's genuinely load-bearing.
- *What I changed:* Rewrote all three definitions so `analysis` requires
  the fact to be the *reason* for the claim ("remove the fact and the claim
  stops holding up"), and made `hot_take` explicitly cover implied verdicts
  like rhetorical questions, not just flatly-stated ones.

**Moment 2**

- *What I asked for:* Reddit's API is blocked for Claude's own tools (no
  direct fetch, no search), so I asked for a way to collect posts anyway.
- *What came back:* A workaround — open Reddit's public `.json` endpoints
  (`/r/soccer/.json`, `/comments/<id>/.json?limit=50`) in my own browser,
  where they aren't blocked, and paste the raw response back in.
- *What I changed:* Adopted that as the actual collection method for every
  batch of posts in this dataset — submissions and comment threads alike —
  rather than hand-copying individual posts one at a time.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure           | Baseline | Trained | Difference |
| ----------------- | -------- | ------- | ---------- |
| Overall accuracy  | 0.656    | 0.656   | +0.000     |
| Macro F1          | 0.264    | 0.264   | +0.000     |
| F1 —`analysis` | 0.000    | 0.000   | +0.000     |
| F1 —`hot_take` | 0.000    | 0.000   | +0.000     |
| F1 —`reaction` | 0.792    | 0.792   | +0.000     |

**What I predicted before I looked:** I can't honestly fill this in. I'd
already seen the trained model's per-label numbers earlier in this session
(while confirming the `device` field for the training-run writeup) before
`baseline.py` was run so there's no genuine blind prediction to record
here, and writing one now, after the fact, would be exactly what this
milestone is designed to catch. Flagging the ordering problem instead of
faking a prediction.

**What the gap actually means:** There is no gap. Both models predict
`reaction` for all 32 held-out posts and nothing else, so every measure
matches to three decimal places. (An earlier run of this table showed the
baseline ahead on the minority labels — 0.296 and 0.250 F1 on `analysis`
and `hot_take` — but that came from a bug in `label_definitions.txt`: the
definitions were hand-wrapped across multiple lines, and `baseline.py`'s
parser reads one label per physical line, so every definition past the
first line-wrap was silently truncated before it ever reached the model.
The "signal" was the zero-shot model reacting to incomplete prompts, not
real taxonomy understanding. Once the definitions were fixed to one line
each, the baseline collapsed to the same always-`reaction` behavior as the
trained model.) With a correctly-specified prompt, neither model
distinguishes `analysis` or `hot_take` from `reaction` on this split at
all — the trained model's accuracy is just the base rate (`reaction` is
134/209 = 64.1% of the training data, and 21/32 = 65.6% of this test
split), and the zero-shot model reaches the identical number by the same
shortcut. Fine-tuning added nothing over a zero-shot model given only the
label definitions.

---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion           | Target                                                | Seed 42                                                        | Seed 7                                                         | Seed 2024                                                      | Verdict                                       |
| ------------------- | ----------------------------------------------------- | -------------------------------------------------------------- | -------------------------------------------------------------- | -------------------------------------------------------------- | --------------------------------------------- |
| 1. Overall accuracy | ≥ 0.80                                               | 0.656                                                          | 0.656                                                          | 0.656                                                          | **MISSED**                              |
| 2. Per-label F1     | every label ≥ 0.50                                   | `analysis` 0.000 · `hot_take` 0.000 · `reaction` 0.792 | `analysis` 0.000 · `hot_take` 0.000 · `reaction` 0.792 | `analysis` 0.000 · `hot_take` 0.000 · `reaction` 0.792 | **MISSED** (`analysis`, `hot_take`) |
| 3. Balance          | no label > 70%; smallest ≥ 30 examples (pre-split)   | `reaction` 64.1%, smallest (`analysis`) 34                 | *(dataset-level — same file for every seed)*                | *(same)*                                                     | **MET**                                 |
| 4. Consistency      | ≥ 75% agreement on staff set                         | *not yet run*                                                |                                                                |                                                                | *pending — run `agreement.py`*           |
| 5. Confidence       | accuracy ≥ 0.85 among predictions ≥ 0.90 confidence | 1.000 (proxy)                                                  | 0.900 (proxy)                                                  | 0.900 (proxy)                                                  | **MET** (on proxy — see note)          |

**Note on row 5:** `results_three_seeds_before.json` only saves confidence in
aggregated thirds (`accuracy_most_confident_third`), not the raw per-prediction
confidence the criterion names (accuracy among predictions ≥ 0.90). The values
above are the most-confident-third accuracy as a substitute, not the literal
metric. Worth reading cautiously either way: since this model predicts
`reaction` for every post (see the confusion matrix below), "confident
predictions are more often right" partly reflects that `reaction` actually is
right 21/32 of the time — a confidence score tracking its own majority-class
bias, not necessarily evidence the model is well-calibrated.

**Note on row 4:** No value yet. `agreement.py` requires `data/staff_labels.csv`,
which TF provides only after `my_staff_labels.csv` is submitted — it
can't be estimated or substituted. This row gets filled in once that file
arrives.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted   | analysis | hot_take | reaction |
| ------------------ | -------- | -------- | -------- |
| **analysis** | 0        | 0        | 5        |
| **hot_take** | 0        | 0        | 6        |
| **reaction** | 0        | 0        | 21       |

**My biggest off-diagonal number, and what it means:**

<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

6 real `hot_take` posts were called `reaction`, and 0 real `reaction` posts
were ever called `hot_take` — entirely one-directional. The same pattern
holds for `analysis`: all 5 real `analysis` posts were called `reaction`,
0 the other way. The model didn't partially learn either boundary and
blur the edges; it never predicts `analysis` or `hot_take` at all, in
either seed's test split. That's a model that fell back entirely on the
majority class rather than one that's confusing two genuinely similar
labels — the direction never reverses, which is what a learned-but-fuzzy
boundary would look like.

---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
| - | --------- | ------ | ------- | ------------- |
| 1 | Overall accuracy | ≥ 0.80, all three seeds | **MISSED** | 0.656 in every seed (42, 7, 2024) — short of 0.80 in each one individually, not just on average |
| 2 | Per-label F1 | every label ≥ 0.50, all three seeds | **MISSED** | `reaction` clears it (0.792 in every seed), but `analysis` and `hot_take` are 0.000 in every seed — the model never predicts either label at all |
| 3 | Balance | no label > 70%; smallest ≥ 30 examples (pre-split) | **MET** | `reaction` is 64.1% of `labels.csv` (< 70%), and the smallest label (`analysis`) has 34 examples (≥ 30) |
| 4 | Consistency | ≥ 75% agreement on staff set | **Not yet measured** | `agreement.py` needs `data/staff_labels.csv`, which hasn't arrived from course staff yet |
| 5 | Confidence | accuracy ≥ 0.85 among predictions ≥ 0.90 confidence | **MET** (on proxy) | Exact threshold isn't saved by the notebook; most-confident-third accuracy (1.000 / 0.900 / 0.900) stands in as a disclosed substitute — see the Run Log note |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

> **This diagnosis is provisional.** Criterion 4 (Consistency) hasn't run yet
> — `data/staff_labels.csv` isn't here — and that's specifically the
> instrument that tells a labelling problem from a model problem. What
> follows is what the other evidence (confusion matrix, split counts, the
> three-seed spread, and the baseline run) can rule in or out on its own; the
> one open question below gets resolved once the agreement report lands.

**Criterion 1 (overall accuracy) and criterion 2 (per-label F1) share one cause: too few examples for `analysis` and `hot_take`.**

How I know, ruling out the other three:

- **Not a genuinely hard label pair.** A hard pair shows up as the model
  sometimes guessing the minority label and getting it half-right — confusion
  in both directions, varying by which posts land in the split. Here
  `analysis` and `hot_take` are at 0.000 F1 in *all three* seeds, with *zero*
  correct predictions and *zero* reverse confusions (`reaction` is never
  mistaken for either of them). The model isn't finding the boundary blurry;
  it isn't looking for the boundary at all.
- **Not a task unreachable from 200 examples, at least not uniformly** — the
  same model, same training setup, same amount of data gets `reaction` to
  0.792 F1. The architecture and the data volume are evidently enough to
  learn *something* here; they just never pay off for the two labels with
  fewer training examples (`analysis` 24, `hot_take` 29, vs. `reaction` 92 in
  the 145-row train split).
- **Consistent with too few examples / class imbalance.** The collapse is
  identical across all three seeds (zero spread on every measure — see the
  Run Log table) despite each seed drawing a different stratified split. That
  seed-invariance is itself evidence: if this were about which specific posts
  happened to land in training, the three seeds would disagree with each
  other at least a little. They don't, which points at something structural
  (the label counts themselves) rather than something that depends on the
  luck of the split. The zero-shot baseline — which never trained on
  `labels.csv` at all — collapses to the identical `reaction`-only behaviour
  given the same test posts and full definitions, which is further evidence
  that the test split's own 21/32 `reaction` majority is enough on its own to
  make "always guess `reaction`" the dominant strategy, independent of
  training.
- **Open question — a boundary applied inconsistently.** What I can't yet
  rule out is whether the `analysis`/`hot_take` scarcity is compounded by
  inconsistent labelling on my part in the original 209 — i.e. whether some
  of those 34 `analysis` and 41 `hot_take` posts are themselves borderline
  calls I made differently in different parts of the file, which would make
  the *effective* signal even thinner than the raw counts suggest. The
  agreement report is exactly the instrument for that, and I'll revise this
  diagnosis once criterion 4 has a real number behind it.

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

**What I changed:** Added 31 new `analysis`-labelled posts collected from
r/soccer comment threads (match post-match threads, an opinion piece on
club ownership, and a news-article thread), nearly doubling `analysis` from
34 to 65 examples. `labels.csv` went from 209 to 240 rows; `hot_take` (41)
is now the thinnest label instead of `analysis`.

**Which diagnosis pointed at it:** Criteria 1 and 2's diagnosis ("too few
examples for `analysis` and `hot_take`," with the other three causes ruled
out — see Diagnoses above) named adding examples to the thinnest label as
the actionable fix, pending the agreement report for the labelling-
consistency question it left open.

### Run Log — Before vs. After

| Criterion | Target | Before — Seed 42 | Before — Seed 7 | Before — Seed 2024 | Before Verdict | After — Seed 42 | After — Seed 7 | After — Seed 2024 | After Verdict |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1. Overall accuracy | ≥ 0.80 | 0.656 | 0.656 | 0.656 | **MISSED** | 0.556 | 0.556 | 0.556 | **MISSED** |
| 2. Per-label F1 | every label ≥ 0.50 | `analysis` 0.000 · `hot_take` 0.000 · `reaction` 0.792 | `analysis` 0.000 · `hot_take` 0.000 · `reaction` 0.792 | `analysis` 0.000 · `hot_take` 0.000 · `reaction` 0.792 | **MISSED** (`analysis`, `hot_take`) | `analysis` 0.000 · `hot_take` 0.000 · `reaction` 0.714 | `analysis` 0.000 · `hot_take` 0.000 · `reaction` 0.714 | `analysis` 0.000 · `hot_take` 0.000 · `reaction` 0.727 | **MISSED** (`analysis`, `hot_take`) |
| 3. Balance | no label > 70%; smallest ≥ 30 examples (pre-split) | `reaction` 64.1%, smallest (`analysis`) 34 | *(same file every seed)* | *(same)* | **MET** | `reaction` 55.8%, smallest (`hot_take`) 41 | *(same file every seed)* | *(same)* | **MET** |
| 4. Consistency | ≥ 75% agreement on staff set | *not yet run* | | | *pending* | *not yet run* | | | *pending* |
| 5. Confidence | accuracy ≥ 0.85 among predictions ≥ 0.90 confidence | 1.000 (proxy) | 0.900 (proxy) | 0.900 (proxy) | **MET** (on proxy) | 0.917 (proxy) | 0.917 (proxy) | 0.917 (proxy) | **MET** (on proxy) |

**Did it help, and how do I know:** No. Nearly doubling `analysis` (34 → 65)
left its F1 at 0.000 in all three seeds, identical to before — the model
still predicts `reaction` for essentially every held-out post (one single
exception: seed 2024 misclassified one `hot_take` post as `analysis`,
instead of as `reaction` like every other minority-label post). Accuracy
and `reaction`'s own F1 both *dropped* (0.656→0.556, 0.792→0.714-0.727),
but that's not a regression — it's the majority-class share shrinking
as the dataset became more balanced (`reaction` 64.1%→55.8% of the file),
and this model's whole strategy is "always guess `reaction`," so its
accuracy just tracks that share. If scarcity were the entire cause, I'd
expect doubling `analysis` to produce at least a few correct predictions
on it — it produced none. That's evidence the collapse isn't (purely) a
data-volume problem, which shifts weight back onto the open question from
the Diagnoses section: a labelling-consistency issue that the agreement
report, once `data/staff_labels.csv` arrives, is the only instrument that
can confirm or rule out.

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
