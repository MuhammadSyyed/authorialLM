# Name That LLM — Competition Brief

**Source:** [https://www.kaggle.com/competitions/name-that-llm](https://www.kaggle.com/competitions/name-that-llm)  
**Tagline:** Identify the model behind the text  
**Fetched / cross-checked:** 2026-10-20 (from Overview, Data, Rules, Leaderboard, and key Discussion threads)

This document is a verified working brief before building a solution. Where pages disagree, or where info is community-only, that is called out explicitly.

---

## 1. One-line task

Build a classifier that, given a short unlabeled text passage, predicts which of **18 frontier LLMs** wrote it (authorship attribution / model fingerprinting).

---

## 2. Problem statement (official)

Ask 18 frontier language models the same question and no two answers read quite the same. Even in a single short paragraph, every model leaves subtle clues in word choice, tone, and sentence structure.

- Standard AI detectors only separate machine vs human text.
- This competition requires **exact source-model** attribution.
- Passages are short (about **40–250 words**), prompts are unseen at test time, and you cannot rely on character-count shortcuts or topic keywords alone.
- Each lab’s training data and post-training leave stylistic fingerprints in vocabulary, phrasing, and punctuation.

**Goal stated by hosts:** show how reliably frontier models can be fingerprinted from short text alone; help audit open benchmarks and understand what makes each model family distinct.

**Topics covered in prompts:** philosophy, physics, mathematics, biology, computer science, and creative prose.

**Providers:** OpenAI, Google, Anthropic, Qwen, ZAI, Moonshot AI (6 labs, 18 models).

---

## 3. Timeline (all times 11:59 PM UTC unless noted)

| Milestone | Date |
| --- | --- |
| Start Date | **October 14, 2026** |
| Entry Deadline (must accept rules) | **January 6, 2027** |
| Team Merger Deadline | **January 6, 2027** |
| Final Submission Deadline | **January 13, 2027** |

Organizers may update the timeline if needed.

---

## 4. Evaluation metric

**Metric:** Balanced accuracy = mean of **per-class recall** over the 18 models.

\[
\text{BalancedAccuracy} = \frac{1}{K} \sum_{k=1}^{K} \frac{|\{i : y_i = k \land \hat{y}_i = k\}|}{|\{i : y_i = k\}|}
\]

where \(K = 18\), \(y_i\) is the true authoring model, and \(\hat{y}_i\) is the prediction.

**Why this metric:** Test passages per model are **not equal**, and the model mix differs between public and private leaderboards. Balanced accuracy gives every model equal weight.

**Chance baseline:** ~\(1/18 \approx 0.0556\).

**Leaderboard split (from public LB page):** Public score uses ~**40%** of test data; final ranking uses the other ~**60%** (private).

### Cross-check: formula rendering

The Overview page’s LaTeX was mangled in HTML fetches (`1K∑…`). The prose definition is unambiguous and matches the standard balanced-accuracy / macro-recall definition above. Community baselines (e.g. `sklearn.metrics.balanced_accuracy_score`) treat it the same way.

---

## 5. Valid labels (exact strings — must match)

Every prediction must be **exactly one** of these 18 names:

| Provider | Model Name |
| --- | --- |
| OpenAI | `gpt-5.5` |
| OpenAI | `gpt-5.6-luna` |
| OpenAI | `gpt-5.6-sol` |
| OpenAI | `gpt-5.6-terra` |
| OpenAI | `gpt-6-astra` |
| Google | `gemini-3.1-pro-preview` |
| Google | `gemini-3.6-flash` |
| Google | `gemini-3.7-flash` |
| Google | `gemini-3.8-flash` |
| Anthropic | `claude-fable-5` |
| Anthropic | `claude-opus-4-6` |
| Anthropic | `claude-opus-4-8` |
| Anthropic | `claude-opus-5` |
| Anthropic | `claude-sonnet-5` |
| Qwen | `qwen3.8-max` |
| ZAI | `glm-5.3` |
| ZAI | `glm-5.3-flash` |
| Moonshot AI | `kimi-k3` |

**Rejection cases:** missing `id`/`model` columns, nulls, duplicate `id`, or any label outside this list.

---

## 6. Dataset

**License:** CC BY-SA 4.0  
**Size (listed):** ~2.14 MB (3 files: `train.csv`, `test.csv`, `sample_submission.parquet`)

### Generation notes (official)

- Every passage is an authentic LLM completion.
- Only light formatting cleanup (e.g. removing a whole-passage `**bold**` wrapper).
- Test prompts do **not** appear in `train.csv`.

### Files

#### `train.csv` — 2,699 passages for 150 prompts

| Column | Type | Description |
| --- | --- | --- |
| `id` | str | Unique anonymised passage id |
| `prompt_id` | str | Anonymised prompt id |
| `text` | str | Generated passage |
| `model` | str | Ground-truth model (one of 18) |

#### `test.csv` — interactive stub + hidden full set

| Column | Type | Description |
| --- | --- | --- |
| `id` | str | Unique anonymised passage id |
| `text` | str | Unlabelled passage to classify |

**No `prompt_id` or `model` in test.**

- Interactive / notebook session: small stub so code runs end-to-end.
- Scoring rerun: full hidden test (~**4,000** passages).

#### `sample_submission.parquet`

| Column | Type | Description |
| --- | --- | --- |
| `id` | str | Matches `test.csv` |
| `model` | str | Predicted label |

### Cross-check: interactive `test.csv` row count (OFFICIAL INCONSISTENCY)

| Source | Claim |
| --- | --- |
| Overview → Submission File | Interactive `test.csv` has a **single** example row |
| Data page | Interactive `test.csv` has **three** example rows |

**Action for building:** write code that handles **any** number of test rows (read all rows from `test.csv`, emit one prediction per `id`). Do not hard-code 1 or 3.

### Cross-check: train structure (community-verified math, consistent with official counts)

Official: **2,699** passages × **150** prompts.

Expected if every prompt had all 18 models: \(150 \times 18 = 2700\).

Community audit ([discussion/750706](https://www.kaggle.com/competitions/name-that-llm/discussion/750706), [discussion/750390](https://www.kaggle.com/competitions/name-that-llm/discussion/750390)):

- **149** prompts have exactly **18** rows (one per model).
- **1** prompt (`p002`) has **17** rows — missing `claude-fable-5`.
- No prompt has a model twice.
- So train is nearly a full “all models answer each prompt once” design.

This is **not** contradicted by the official count (2699 = 2700 − 1).

### Cross-check: test structure (partially open)

Official facts that are solid:

- Test prompts are **disjoint** from train prompts.
- Per-model counts are **unequal**.
- Public vs private model mix **differs**.
- Hidden size ~**4000**.

Community experiments (not host-confirmed): hard “one model per prompt cluster” / Hungarian assignment **helps grouped CV** but **hurts public LB** (e.g. 0.353 → 0.344). Suggests hidden test may have **few passages per prompt**, so treat assignment tricks carefully. Host has not answered [discussion/750768](https://www.kaggle.com/competitions/name-that-llm/discussion/750768) yet (whether multiple models share a prompt in test; whether joint inference across test rows is allowed).

---

## 7. Submission format (Code Competition)

This is a **Code Competition**. Notebooks must:

1. Read: `/kaggle/input/competitions/name-that-llm/test.csv`
2. Write: `submission.parquet` in the notebook working directory
3. Include every test `id` exactly once, with columns `id` and `model`

Minimal pattern (official):

```python
import pandas as pd

test = pd.read_csv('/kaggle/input/competitions/name-that-llm/test.csv')
submission = pd.DataFrame({'id': test['id'], 'model': predictions})
submission.to_parquet('submission.parquet', index=False)
```

### Cross-check: CSV example vs parquet requirement

Overview shows a CSV-looking header (`id,model`) for illustration, but the required artifact is **`submission.parquet`**. Always write parquet.

---

## 8. Code / compute requirements

| Constraint | Value |
| --- | --- |
| Submission method | Kaggle Notebooks only |
| CPU notebook runtime | ≤ **9 hours** |
| GPU notebook runtime | ≤ **9 hours** |
| Internet at scoring | **Disabled** |
| External data | Freely & publicly available external data allowed, including pre-trained models |
| Output filename | Must be `submission.parquet` |

Training may happen outside Kaggle; attach weights/models as datasets for offline inference.

---

## 9. Prizes

**Total prize pool:** $50,000  

| Place | Prize |
| --- | --- |
| 1st–10th | **$5,000 each** |

Also awards Kaggle points & medals.

**Winner license (Rules):** CC-BY 4.0 / open-source obligations for winning submissions (OSI-approved license that does not limit commercial use; winners must deliver reproducible training + inference code and documentation).

**Data access license (Rules):** CC BY-SA 4.0.

---

## 10. Rules highlights (competition-specific)

| Rule | Detail |
| --- | --- |
| Max team size | **5** |
| Submissions per day | **5** |
| Final submissions selectable | **2** |
| Team mergers | Allowed until merger deadline; combined submission count must stay within allowed max |
| Private code sharing | Forbidden outside teams |
| Public code sharing | OK on competition forums/notebooks (OSI-approved license) |
| External data | Publicly available & equally accessible at no cost, **or** meets “Reasonableness” cost/accessibility standard |
| Hand-labeling / human prediction of test | Forbidden |
| Eligibility | 18+ (or majority age); not from sanctioned regions; one Kaggle account |
| Under-18 | Consent forms required (see host thread); under 13 not allowed |

### Cross-check: Host vs Sponsor naming

| Source | Says |
| --- | --- |
| Overview sidebar | Competition Host: **Kaggle** |
| Official Rules | Competition Sponsor: **Google LLC** (1600 Amphitheatre Parkway, Mountain View, CA 94043) |
| Overview prose | “hosted by Kaggle” |
| Authors / citation | Yao Yan, Martyna Plomecka, Ashley Oldacre (Kaggle Staff active on forums) |

**Interpretation:** Platform/hosting is Kaggle; legal sponsor in Rules is Google LLC. Authors of the competition page: Yao Yan, Martyna Plomecka, Ashley Oldacre.

### Open rule question (as of fetch date)

Participants asked whether **privately generating** extra passages via paid model APIs is allowed under the Reasonableness Standard, and whether **already-public** API-generated Kaggle datasets are fine ([discussion/749613](https://www.kaggle.com/competitions/name-that-llm/discussion/749613), [discussion/749447](https://www.kaggle.com/competitions/name-that-llm/discussion/749447)). **No clear host ruling** was present in the fetched threads. Safest documented path until clarified: use **public, free-to-all** datasets/models that all participants can access.

---

## 11. Participation snapshot (at fetch time)

| Metric | Count |
| --- | --- |
| Entrants | 7,053 |
| Participants | 2,141 |
| Teams | 2,062 |
| Submissions | 11,834 |

**Public leaderboard (top, ~40% test):** leading scores around **0.66** (e.g. “Hod Carriers” ~0.666). Scores move quickly; treat as a snapshot only.

Official starter notebook: [Starter Notebook for Name That LLM](https://www.kaggle.com/code/martynaplomecka/starter-notebook-for-name-that-llm-competition) (MartynaPlomecka / Yao Yan) — public score ~**0.322** on early version.

---

## 12. Community findings useful for building (not official)

Mark these as **community evidence**, useful for planning but not host guarantees.

### Validation

- Use **`StratifiedGroupKFold` grouped by `prompt_id`**. Random K-fold can be badly misleading (reported ~0.22 vs ~0.37 while LB ~0.35).
- Reason: same-prompt passages from other models leak topic cues into random folds.

### Strong CPU baseline

- Char TF-IDF (1–5) + word TF-IDF (1–2), `lowercase=False`, LogisticRegression.
- Reported: grouped CV ~**0.34**, public LB ~**0.35** ([discussion/750390](https://www.kaggle.com/competitions/name-that-llm/discussion/750390)).

### Difficulty structure

- **Short passages (<~60 words)** much harder than long ones.
- Many errors are **right lab, wrong sibling model** (~55–58% of mistakes in style-only models).
- Punctuation separates **labs** better than **siblings** (e.g. Claude/GLM heavy em-dashes; GPT/Gemini curly apostrophes; Gemini nearly zero em-dashes in train — may not hold on external/proxy data).
- GLM is stylistically close to Claude → hard confusion pair.

### LLM fine-tuning (community)

- Qwen3-class LoRA classifiers reported mid-to-high **0.4x** LB without extra data; stronger public notebooks higher.
- Inference can fit free **2×T4** (use fp16, not bf16; merge LoRA; batch by length).
- No internet on submit → attach base models/weights as Kaggle datasets.

### Extra data

- Public corpora exist (18-model wide corpus, Claude-only API passages, etc.).
- Mismatched formatting (line breaks, Markdown, length) can **hurt** if naively mixed into train.
- Targeted use (e.g. Claude-only gate) reported more helpful than dumping into 18-way training.

### Test-time prompt clustering / assignment

- Helps CV simulating full 18-per-prompt; **often hurts LB** → do not bank on hard assignment without LB confirmation.

### Reconstructed prompts (community, not ground truth)

- [discussion/749455](https://www.kaggle.com/competitions/name-that-llm/discussion/749455): LLM-reconstructed instructions for 150 train `prompt_id`s; style often “explain X vividly,” one self-contained paragraph, random target length ~40–250 words.

---

## 13. Getting started (official host resources)

- Host welcome / Discord pointer: [discussion/749382](https://www.kaggle.com/competitions/name-that-llm/discussion/749382) (Ashley Oldacre, Kaggle Staff)
- Discord is **public** for competition chat; **do not** privately share code/data there. Staff do **not** monitor Discord for official answers — keep important Q&A on Kaggle forums.

---

## 14. Citation (official)

> Yao Yan, Martyna Plomecka, and Ashley Oldacre. Name That LLM. https://www.kaggle.com/competitions/name-that-llm, 2026. Kaggle.

---

## 15. Cross-check summary (before we build)

| Item | Status | Notes |
| --- | --- | --- |
| Task = 18-way model authorship from short text | Verified | Overview + Data |
| Metric = balanced accuracy (macro recall) | Verified | Overview prose; formula reconstructed |
| Labels = exact 18 strings | Verified | Overview table |
| Train = 2699 / 150 prompts | Verified | Data page; math matches 2700−1 |
| Train ≈ one row per (prompt, model) | Verified (community + counts) | Missing one `claude-fable-5` on `p002` |
| Test prompts unseen; ~4000 hidden | Verified | Overview + Data |
| Interactive test stub size 1 vs 3 | **Conflict** | Handle N rows dynamically |
| Submit `submission.parquet` via Notebook, no internet | Verified | Overview Code Requirements |
| 9h CPU/GPU, 5/day, team ≤5, 2 finals | Verified | Overview + Rules |
| Prize $50k ($5k × top 10) | Verified | Overview + Rules |
| Sponsor Google LLC vs Host Kaggle | Naming difference | Both stated; Rules win for legal sponsor |
| Public ~40% / private ~60% LB | Verified | Leaderboard page |
| External API self-gen allowed? | **Unresolved** | Prefer public free datasets until host clarifies |
| Test has multi-model-per-prompt? | **Unresolved** | Hard assignment hurt LB in community tests |
| Passages ~40–250 words, light cleanup only | Verified | Overview + Data |

---

## 16. Practical build checklist (derived from verified facts)

1. Accept rules on Kaggle before Entry Deadline.
2. Load `train.csv`; validate 18 labels and ~2699 rows.
3. Validate with **prompt-grouped** CV + balanced accuracy.
4. Keep raw text (don’t “clean away” punctuation/case carelessly).
5. Train offline if needed; ship a **no-internet** inference notebook writing `submission.parquet`.
6. Ensure predictions are exact label strings; one row per test `id`.
7. Prefer public external assets; track licenses (CC BY-SA competition data; winner OSS obligations).
8. Watch short-passage performance and within-lab confusions (Claude↔GLM, GPT siblings, Gemini siblings).

---

*End of brief. Ready to design an approach once you confirm priorities (CPU baseline first vs LoRA, external data policy, etc.).*
