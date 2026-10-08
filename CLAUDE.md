# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a student starter/submission repo for an AI201 course assignment ("TakeMeter"): design a
label taxonomy for a text community, hand-label ~200 posts, fine-tune a small transformer classifier
on them, then (unit 6) stress-test it across seeds, compare it to a zero-shot baseline, and check
label consistency against a staff-labelled set. `README.md` *is* the submission — it ships as a
template with HTML comments describing what each section needs; filling it in is part of the work.
Everything runs locally (no API keys, no hosted services) via a Jupyter notebook and two CLI scripts.

Read `RUNNING.md` for the full walkthrough — it's the canonical doc for setup, the notebook, and
troubleshooting. This file only covers what's relevant to editing/running code in the repo.

## Commands

```bash
# Setup
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\Activate.ps1
pip install -r requirements.txt
python test.py                     # environment/dependency check, not graded

# unit 6 — zero-shot baseline (reads label_definitions.txt + test_split.csv)
python baseline.py
python baseline.py --trained results.json   # adds the Baseline vs. Trained table
python baseline.py --help

# unit 6 — label-consistency check against the staff set
python agreement.py
python agreement.py --help
```

Python must be 3.11, 3.12, or 3.13 (not 3.14 — pinned stack doesn't support it). `test.py` encodes
all the pins and hardware checks (disk/RAM/CUDA-MPS-CPU detection); don't duplicate that logic
elsewhere, just run it.

There is no automated test suite for correctness in the usual sense — `test.py`, `baseline.py`, and
`agreement.py` are the three runnable scripts, and the notebook is run interactively, cell by cell,
not executed headlessly. When editing `baseline.py` or `agreement.py`, the `score`/`compare` functions
are pure (no model calls) and are the part worth unit-testing if you add tests.

## Architecture

### The pipeline, end to end

1. **`labels.csv`** (student-created, gitignored nowhere, one unsplit file) — the ~200 labelled posts.
2. **`takemeter.ipynb`**, sections 1–5 (unit 5): reads `labels.csv`, validates it (`check_dataset` —
   catches unknown labels, >70% dominance, duplicates, thin labels), does a stratified 70/15/15 split
   (`split_dataset`, seeded), fine-tunes `BASE_MODEL` (`distilbert-base-uncased` by default) via a HF
   `Trainer` (`train_model`), evaluates on the held-out test split (`evaluate` — also computes a
   confidence report), and writes `results.json` + `test_split.csv`.
3. **`takemeter.ipynb`**, sections 6–7 (unit 6): reruns the full split→train→evaluate loop for
   `SEEDS = [42, 7, 2024]`, writes `results_three_seeds_<RUN_LABEL>.json` (`RUN_LABEL` is `"before"`
   or `"after"` — the student flips it manually before the post-improvement rerun, or it silently
   overwrites the baseline evidence file), then prints a spread table and a markdown confusion matrix.
4. **`baseline.py`** (unit 6, standalone): zero-shot-classifies the *same* held-out posts
   (`test_split.csv`, for a like-for-like comparison with the trained model) using
   `facebook/bart-large-mnli`, scored only against `label_definitions.txt` (the label definitions are
   the entire prompt — no training, no examples). Writes `baseline_results.json` and prints the
   Baseline vs. Trained markdown table for the README.
5. **`agreement.py`** (unit 6, standalone): compares the student's own labels on a 30-post staff set
   (`data/staff_posts.csv` + `data/staff_labels.csv`, joined on `id`, under the taxonomy in
   `data/staff_taxonomy.md`) against the student's `my_staff_labels.csv`. Posts are matched by
   normalized text, not row order — a retyped/trimmed post silently fails to match. Writes
   `agreement_results.json` and a disagreement table; the script deliberately stops at arithmetic and
   leaves adjudication (who's right and why) to the README.

### Why the file layout is the way it is

- `results.json`, `test_split.csv`, `results_three_seeds_*.json`, `baseline_results.json`, and
  `agreement_results.json` are **evidence artifacts** and are intentionally *not* in `.gitignore`
  (see the note at the bottom of `.gitignore`) — they get committed, not regenerated silently.
- `label_definitions.txt` and `labels_template.csv` are the student-facing *shape* of two inputs
  (`baseline.py`'s prompt source and `labels.csv`'s column format, respectively) — not sample data.
- `data/practice_labels.csv` is in-class practice data (lets the notebook run before real data
  exists); `data/staff_posts.csv` + `data/staff_taxonomy.md` ship with the starter, `data/staff_labels.csv`
  is supplied later by course staff and is required for `agreement.py` to run.
- `criteria.md` and `README.md` are prose deliverables (acceptance criteria written *before*
  training, and the full write-up respectively) — editing them is part of the assignment, not
  boilerplate to clean up.

### Cross-cutting invariants worth knowing before changing code

- `LABELS` in the notebook must match the `label` column of `labels.csv` **exactly** (case/spelling) —
  this is the most common break (`check_dataset` catches it).
- The notebook must be run with this project's `.venv` as the kernel and from inside this repo
  directory — it resolves paths relative to its own cwd and will misbehave from a copy.
- `baseline.py` and `agreement.py` both reconfigure stdout/stderr to UTF-8 on import because Windows
  consoles default to a codepage that can't encode the em-dashes/arrows used in their output —
  preserve that if touching the top of either file.
- `baseline.py` falls back to a random sample of `labels.csv` (not `test_split.csv`) if the held-out
  file is missing, and prints a warning that the comparison isn't like-for-like — don't treat that
  fallback path as equivalent to the real one.
