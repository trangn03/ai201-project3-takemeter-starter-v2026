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

**Moment 3**

- *What I asked for:* After the before-run collapsed to predicting `reaction`
  on every test post, I asked Claude to read the confusion matrix and tell me
  whether `analysis` and `hot_take` were being confused *with each other*
  (a genuinely hard pair) or something else.
- *What came back:* Neither minority label ever received a single prediction
  in either direction — every `analysis` and `hot_take` post was routed to
  `reaction`, and `reaction` was never misrouted the other way. That
  one-directional pattern (not mutual confusion) pointed at scarcity/imbalance
  rather than the labels being conceptually too close together — a
  genuinely-hard-pair failure would show posts leaking both ways.
- *What I changed:* Picked "add more `analysis` examples" as the one
  improvement to try, on the strength of that diagnosis. After the after-run
  came back with `analysis`/`hot_take` F1 still at 0.000 despite nearly
  doubling `analysis` (34→65), I asked Claude to re-read the new confusion
  matrix rather than assume the fix just hadn't been enough — it's almost
  the same one-directional pattern as before (seed 2024 is the one
  exception: 1 of 6 `hot_take` posts landed on `analysis` instead of
  `reaction`). That's why "What's Still Broken" points past pure data
  volume and back toward the open labelling-consistency question, instead
  of recommending a second, larger data-addition round.

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
| 4. Consistency      | ≥ 75% agreement on staff set                         | 93.3% (28/30)                                                | *(dataset-level — same staff set regardless of seed)*         | *(same)*                                                      | **MET**                                 |
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

**Note on row 4:** Measured once `data/staff_labels.csv` arrived — 28/30
posts matched staff's labels under their taxonomy (`data/staff_taxonomy.md`),
clearing the ≥75% target. Both disagreements are adjudicated in the
Agreement Report section below.

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
| 4 | Consistency | ≥ 75% agreement on staff set | **MET** | 28/30 = 93.3% against `data/staff_labels.csv`; both disagreements traced to the written rule applied loosely (pile (a)), not a gap or a defensible alternate reading — see the Agreement Report below |
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

**Agreement rate:** 28 / 30 = 93.3%

**Disagreements**

**1.**

> *The post:* "Times are down about 3% since the shoe rules changed, so
> obviously the shoes are the whole story and the sport has a problem."
>
> *Staff said / I said:* Staff: `hot_take` · I said: `analysis`
>
> *My call, and why:* Staff is right. This is almost exactly the taxonomy's
> own worked example under Rule 1 ("times are down 3% since 2017" →
> `hot_take`, "the 3% is offered as a flourish, not as the reason"). "So
> obviously... the whole story" shows the conclusion was already reached
> before the stat was introduced — the 3% is decoration, not the thing doing
> the arguing. I pattern-matched "has a stat → `analysis`" instead of
> checking whether the stat was load-bearing.
>
> *Which pile:* (a) — the rule covered this explicitly (it's the canonical
> Rule 1 case), I applied it loosely.

**2.**

> *The post:* "Signed up. Card's charged, no going back, immediately regret
> everything."
>
> *Staff said / I said:* Staff: `reaction` · I said: `hot_take`
>
> *My call, and why:* Staff is right. Rule 2's test is whether the claim is
> about the writer or about the world. Every clause here is first-person and
> about something that just happened to the writer (signed up, card charged,
> regret) — there's no general claim about the world. The declarative tone of
> "no going back" read as hot-take-style confidence to me, but tone isn't the
> test; subject is, and the subject is entirely the writer's own immediate
> experience.
>
> *Which pile:* (a) — the rule covered this, the confident phrasing pulled me
> toward `hot_take` despite the post failing the "about a claim vs. about the
> writer" test.

**What the pattern in my disagreements tells me:** Both misses are pile (a)
— a written rule applied loosely, not a gap in the definitions or a
defensible alternate reading — and both came from the same failure mode:
reacting to a surface feature (a stat being present, a confident tone)
instead of running the actual test the rule specifies (is the fact
load-bearing; is the claim about the writer or the world). That said, it's 2
slips in 30, a 93.3% rate well clear of the ≥75% target, so this doesn't read
as pervasive inconsistency — it argues against my own labelling being a
major contributor to the `analysis`/`hot_take` collapse in the trained
model, which narrows rather than confirms the "boundary applied
inconsistently" open question from the Diagnoses section above.

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
| 4. Consistency | ≥ 75% agreement on staff set | 93.3% (28/30) | *(same measurement — doesn't depend on model or seed)* | *(same)* | **MET** | 93.3% (28/30) | *(same)* | *(same)* | **MET** |
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

**Criterion 1 — Overall accuracy (still MISSED, 0.556 after, vs. ≥ 0.80
target):** Accuracy can't clear 0.80 while two of three labels get 0 recall,
so this is downstream of criterion 2, not a separate problem. What I'd do
next: nothing further until criterion 2 moves, since fixing accuracy
directly without fixing the per-label collapse would just mean juicing the
majority class. Why I stopped: this unit's improvement budget was one
iteration (add examples to the thinnest label), and I used it; a second
iteration is out of scope for this submission.

**Criterion 2 — Per-label F1 (still MISSED, `analysis`/`hot_take` still
0.000 after doubling `analysis`):** What I'd do next: (a) run `agreement.py`
once `data/staff_labels.csv` arrives and use it to check whether some of
the 65 `analysis` / 41 `hot_take` posts are themselves inconsistently
labelled, which would explain why more raw examples didn't help; (b) try
class-weighted loss or oversampling so training can't just default to
`reaction`; (c) if neither of those explains it, add a second, larger batch
of `analysis`/`hot_take` examples and re-test, since 65/41 may still not be
enough. Why I stopped: ran out of time inside this unit after running the
one data-addition experiment and seeing it fail cleanly — the honest
finding (scarcity alone doesn't explain it) was itself worth reporting
rather than papering over with an untested second fix.

**Criterion 4 — Consistency (still *pending*, not MISSED):** What I'd do:
run `agreement.py` the moment `data/staff_labels.csv` is supplied and fold
the disagreement adjudication into the Diagnoses section above, since it's
the only instrument that can confirm or rule out the labelling-consistency
explanation the improvement result (criterion 2) now points back toward.
Why I stopped: this is blocked on an external dependency (the TF hasn't
provided the staff labels yet), not a time or effort tradeoff — there's
nothing to substitute it with per the assignment's own instructions.

**The gap between what I meant my labels to capture and what the model
learned:**

<!-- Two sentences. Your confusion matrix is the evidence. -->

My labels are meant to separate posts by whether a specific fact is
load-bearing for a claim — a structural, reasoning-dependent distinction.
The confusion matrix shows the model learned no such structure at all: every
single `analysis` (10/10) and `hot_take` (6/6) post in the after test split
(seed 42) was routed to `reaction`, with zero predictions ever landing on
either minority label in either direction — it learned to default to the
majority surface pattern, not the fact-dependency the taxonomy actually
tests for.

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
