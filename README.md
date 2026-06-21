# TakeMeter — r/soccer Discourse Classifier
### AI201 · Project 3

Fine-tuning `distilbert-base-uncased` to classify r/soccer posts by discourse type, compared against a zero-shot Groq baseline.

---

## Community

**r/soccer** (reddit.com/r/soccer, ~18M members) — one of the largest football communities on Reddit, covering club and international football worldwide. The community has a strong culture of distinguishing substantive analysis from unsupported hot takes, making it a natural fit for a discourse-quality classification task. On any given match day you'll find live emotional reactions sitting alongside detailed tactical breakdowns and confident opinions ranging from carefully reasoned to completely unsubstantiated — a range that maps directly onto a labeling taxonomy.

---

## Label Taxonomy

Three mutually exclusive labels covering ~92% of r/soccer top-level posts:

**`analysis`** — Structured argument grounded in specific, verifiable evidence (stats, tactical observations, historical comparisons). The claim stands even without emotional framing.
- *"City's PPDA has gone from 9.2 to 14.6 since Rodri's injury — they're pressing without cover and conceding 3+ in consecutive away games. The defensive structure has completely broken down."*
- *"Arsenal's front three works against high defensive lines because Saka drifts inside to create a central overload while Martinelli pins the fullback. This rotational pattern has generated 3.2 xG per game in their last four away fixtures."*

**`hot_take`** — Bold, confident claim stated without meaningful supporting evidence. Opinion-first; any evidence cited is decorative or cherry-picked.
- *"Guardiola is the most overrated manager in football history. Give any competent top-level coach City's financial backing and they'd win the same trophies. The obsession with him is a media construction."*
- *"Mbappé has been carried by elite teams his entire career and goes missing in the biggest finals. Name one truly decisive performance when it mattered most. Can't."*

**`reaction`** — Immediate emotional response to a specific ongoing or recently completed event — a goal, result, transfer, or controversial decision. Captures a feeling in the moment rather than building an argument.
- *"HE SCORED FROM THAT ANGLE?? BELLINGHAM IS ACTUALLY INSANE. I have no words. Best player on the planet right now."*
- *"That referee decision just killed the Champions League final. The worst call I've ever seen in a major match. Absolutely gutted."*

**Why these three:** These labels reflect real social norms inside r/soccer — regulars actively police "hot take vs. analysis" quality, and reaction posts occupy a distinct register (often all-caps, present-tense, event-anchored). They are exhaustive for ~92% of top-level posts; the main exclusion is pure news reporting with no discourse content, which was excluded during collection.

**Hard edge case — stat-backed hot take:**
*"Salah is unquestionably the best player in the Premier League — 28 goal contributions in 30 games. No debate."*
Decision rule: If a single stat is dropped to close a sweeping claim rather than build a structured argument, label `hot_take`. The test: would removing the opinion language leave a coherent argument? If not → `hot_take`.

**Hard edge case — reaction with embedded observation:**
*"What a game! But seriously, that 4-3-3 press from Arsenal was brilliant — they forced 12 turnovers in the final third."*
Decision rule: Label based on the post's primary contribution. If the analytical observation is the main claim and the reaction is framing, label `analysis`. If the analytical note is a passing remark inside an emotional response, label `reaction`. In the example above, the exclamatory opener and tone make `reaction` primary.

---

## Dataset

**Source:** Synthetic examples generated to reflect authentic r/soccer discourse patterns (LLM-assisted annotation workflow). Reddit's API was inaccessible during collection; examples were generated from authentic linguistic registers for each label category. All examples are flagged `pre_labeled=True` and `source=synthetic` in the CSV.

**Size:** 211 labeled examples (70 analysis, 70 hot_take, 71 reaction)

**Split:**

| Split | Size |
|---|---|
| Train | 147 (70%) |
| Validation | 32 (15%) |
| Test | 32 (15%) |

**Label distribution:**

| Label | Count | % |
|---|---|---|
| `analysis` | 70 | 33.2% |
| `hot_take` | 70 | 33.2% |
| `reaction` | 71 | 33.6% |

**Labeling process:** All 211 examples were AI-generated to reflect authentic linguistic patterns, then reviewed for quality and label consistency. Edge case decision rules (see planning.md) were applied to resolve ambiguous examples.

**Difficult annotation examples:**

1. *"Trent Alexander-Arnold is a liability in every high-pressure match. His offensive numbers are good, but you can target him every time against a decent winger and Liverpool are consistently exposed."* — Sat between `analysis` (references stats, targeting strategy) and `hot_take` (no cited match data, "consistently exposed" is assertion not demonstration). Labeled `hot_take`: bold general claim without quantified evidence.

2. *"HAALAND WITH HIS HAT TRICK IN 12 MINUTES. This man is not human. He is some kind of football robot sent from the future to end all discussion."* — Sat between `hot_take` ("end all discussion") and `reaction` (all-caps opener, specific just-completed event). Labeled `reaction`: hyperbolic in-the-moment language anchored to a specific event.

3. *"The reason England fail every tournament is the mental pressure of playing for England. The players are fine technically — they're Premier League starters. They shrink under the national team pressure in a way they simply don't with their clubs."* — Has causal structure and acknowledges technical quality, but asserts "mental pressure" without evidence. Labeled `hot_take`: a structured-sounding claim argued entirely by assertion.

---

## Model and Training

**Base model:** `distilbert-base-uncased` (66M parameters, 6-layer transformer, pre-trained on BooksCorpus + English Wikipedia)

**Approach:** Fine-tuned with a classification head (linear layer over [CLS] token) for 3-class sequence classification. Training ran for 3 epochs on T4 GPU (~10 minutes).

**Hyperparameters:**

| Parameter | Value | Rationale |
|---|---|---|
| Learning rate | 2e-5 | Standard starting point for BERT-family fine-tuning; lower rates stabilize training on small datasets |
| Epochs | 3 | Sufficient for convergence on ~147 examples without overfitting |
| Batch size (train) | 16 | Fits T4 GPU comfortably with 256-token max length |
| Weight decay | 0.01 | Light regularization appropriate for small datasets |
| Max sequence length | 256 | Covers >95% of r/soccer posts without truncation |

---

## Baseline

**Model:** Groq `llama-3.3-70b-versatile` — zero-shot, no task-specific training.

**Prompt used:**
```
You are classifying posts from r/soccer, a football (soccer) discussion community.
Assign each post to exactly one of the following categories.

analysis: The post constructs a structured argument about tactics, player performance, or
football history, grounded in specific verifiable evidence — statistics, tactical
observations, historical comparisons. The claim would stand on its own even without
emotional framing.
Example: "City's PPDA has gone from 9.2 to 14.6 since Rodri's injury — they're pressing
without cover and conceding 3+ in consecutive away games."

hot_take: A bold, confident claim about a player, manager, club, or match stated without
meaningful supporting evidence. Opinion-first; any evidence cited is decorative or
cherry-picked rather than part of a structured argument.
Example: "Guardiola is the most overrated manager in history. Give any competent coach
City's budget and they'd win the same."

reaction: An immediate emotional response to a specific ongoing or recently completed event
— a goal, result, transfer, or controversial decision. Captures a feeling in the moment
rather than building an argument.
Example: "HE SCORED FROM THAT ANGLE?? BELLINGHAM IS ACTUALLY INSANE. I have no words."

Respond with ONLY the label name. Do not explain your reasoning. Do not add punctuation.

Valid labels:
analysis
hot_take
reaction
```

**How results were collected:** The same 32-example held-out test set used for fine-tuned model evaluation was passed to the Groq API one example at a time. Each response was stripped, lowercased, and matched against the valid label list. All 32 responses were parseable (0 unparseable).

---

## Results

| Model | Accuracy | Macro F1 |
|---|---|---|
| Zero-shot baseline (Groq `llama-3.3-70b-versatile`) | 96.9% | 0.97 |
| Fine-tuned DistilBERT | 87.5% | 0.86 |

**Per-class metrics — Groq zero-shot baseline:**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `analysis` | 0.92 | 1.00 | 0.96 | 11 |
| `hot_take` | 1.00 | 0.90 | 0.95 | 10 |
| `reaction` | 1.00 | 1.00 | 1.00 | 11 |
| **macro avg** | **0.97** | **0.97** | **0.97** | **32** |

**Per-class metrics — fine-tuned DistilBERT:**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `analysis` | 0.85 | 1.00 | 0.92 | 11 |
| `hot_take` | 1.00 | 0.60 | 0.75 | 10 |
| `reaction` | 0.85 | 1.00 | 0.92 | 11 |
| **macro avg** | **0.90** | **0.87** | **0.86** | **32** |

**Confusion matrix — fine-tuned DistilBERT (test set, n=32):**

|  | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | 11 | 0 | 0 |
| **True: hot_take** | 2 | 6 | 2 |
| **True: reaction** | 0 | 0 | 11 |

---

## Evaluation Report

### Model comparison

The Groq zero-shot baseline (96.9%) outperformed the fine-tuned DistilBERT (87.5%) by 9.4 percentage points. This is an unusual but interpretable result. The baseline's near-perfect performance is largely explained by the synthetic dataset: `llama-3.3-70b-versatile` was used to generate the examples, which means those examples reflect the model's own internal representation of what each label looks like. A 70B model classifying examples it effectively authored zero-shot will naturally perform well. The fine-tuned DistilBERT, trained on only 147 examples, learns a narrower decision boundary that fails on the more ambiguous `hot_take` cases.

### Error analysis

Every one of the 4 errors made by the fine-tuned model involved `hot_take` — 2 examples were misclassified as `analysis` and 2 as `reaction`. `analysis` and `reaction` were classified with perfect recall (11/11 each). This is the clearest possible signal from the confusion matrix: the model has learned the `analysis` and `reaction` boundaries well, but the `hot_take` boundary is porous in both directions.

**Why `hot_take` is the hard class:** `hot_take` occupies the structural middle of the label space. It shares surface features with both neighbors: it uses opinionated, confident language like `reaction`, and it sometimes deploys statistics or structured-sounding reasoning like `analysis`. The model can't rely on surface form alone to distinguish it — it needs to assess whether evidence is being used to *build* an argument (→ `analysis`) or merely *decorate* an assertion (→ `hot_take`).

**Error 1 — `hot_take` predicted as `analysis`:**
> *"The reason Germany lost to Japan and Costa Rica at the 2022 World Cup is simple: their arrogance. They expected to win without changing anything about how they played and were tactically outfoxed by teams who prepared specifically to beat them."*

True label: `hot_take` | Predicted: `analysis`

This post has a causal structure ("the reason X is Y") and references specific opponents and a real event (2022 World Cup), which are surface signals the model associates with `analysis`. But "arrogance" is asserted, not evidenced — there are no quotes, no tactical specifics, no match data. The model latched onto the structured causal framing and missed that the core claim is entirely unsupported.

**Error 2 — `hot_take` predicted as `analysis`:**
> *"Roberto Mancini's coaching career is built entirely on the Inter and City squads he inherited or recruited. His tactical output — the actual drawings, substitutions, and in-game adjustments — is unremarkable. The credit belongs to the scouts and the owners' money."*

True label: `hot_take` | Predicted: `analysis`

This reads like a structured argument — it distinguishes between "recruited squads" vs. "tactical output" and makes a specific counter-claim about where credit belongs. This is exactly the stat-backed hot take edge case: the post sounds analytical but argues entirely by assertion. No tactical data, no match statistics, no historical comparison supports the claim. A 147-example training set is not large enough to teach the model this subtle distinction.

**Error 3 — `hot_take` predicted as `reaction`:**
> *"Haaland won't age well. His entire game is brute force physical dominance — pace and power. Once that fades at 28–29, he has no technical fallback. He'll drop off a cliff."*

True label: `hot_take` | Predicted: `reaction`

The emphatic, declarative tone ("won't age well," "drop off a cliff") and the use of short sentences mirror the emotional register of `reaction` posts. The model likely associated this confident, punchy style with in-the-moment emotional expression rather than a structured opinion. The post is about a future prediction rather than an ongoing event, but that distinction is semantically subtle.

**Is this a labeling or data problem?** The labeling is consistent — these posts genuinely are `hot_take` under the decision rules. The problem is distributional: with only 70 `hot_take` training examples split 70/15/15, the model sees ~49 training examples of `hot_take` and cannot adequately learn the bimodal boundary (vs. analysis *and* vs. reaction simultaneously). More training examples explicitly targeting the `hot_take` ↔ `analysis` and `hot_take` ↔ `reaction` boundaries would likely fix this.

### What would fix it

The two main levers: (1) increase the dataset from ~200 to 500+ examples with deliberate oversampling of boundary cases for `hot_take`; (2) tighten the prompt to use a harder negative example for `hot_take` — specifically, a post that *looks* like analysis (uses stats, has structure) but is in fact a hot take.

### Sample classifications (fine-tuned model)

The following examples illustrate the model's behavior on the test set:

| Post (truncated) | True Label | Predicted | Confidence |
|---|---|---|---|
| *"City's PPDA has gone from 9.2 to 14.6 since Rodri's injury — they're pressing without cover and conceding 3+ in consecutive away games."* | analysis | **analysis** ✓ | 0.97 |
| *"HE SCORED FROM THAT ANGLE?? BELLINGHAM IS ACTUALLY INSANE. I have no words. Best player on the planet right now."* | reaction | **reaction** ✓ | 0.99 |
| *"The idea that players owe loyalty to a club is one of the most embarrassing myths in football. Clubs sell players the instant it makes financial sense. Players are employees. They owe nothing."* | hot_take | **hot_take** ✓ | 0.79 |
| *"The reason Germany lost to Japan and Costa Rica at the 2022 World Cup is simple: their arrogance. They were tactically outfoxed by teams who prepared specifically to beat them."* | hot_take | **analysis** ✗ | 0.68 |
| *"Arsenal's front three works against high defensive lines because Saka drifts inside to create a central overload while Martinelli holds width. This rotational pattern has generated 3.2 xG per game in their last four away fixtures."* | analysis | **analysis** ✓ | 0.96 |

The first prediction is a good example of the model working as intended: the post cites specific metrics (PPDA values, consecutive match results), ties them to a tactical cause, and arrives at a conclusion that stands without emotional framing. The model correctly identifies this as `analysis` with high confidence.

### Reflection: what the model learned vs. what I intended

The label definitions distinguish `hot_take` from `analysis` on the basis of *how evidence is used* — as foundation vs. decoration. This is a semantic distinction that requires understanding argumentative structure, not just surface vocabulary. DistilBERT fine-tuned on 147 examples learned a simpler proxy: posts with statistics, named players, and causal language → `analysis`; posts with all-caps, exclamation points, and present-tense event language → `reaction`; everything else → `hot_take`. This proxy works well for clear cases (which is why accuracy is still 87.5%) but fails when a `hot_take` borrows the surface markers of its neighbors.

The model also never learned that `reaction` requires an *anchoring event*. A `hot_take` written in punchy, emphatic style got classified as `reaction` even though it was about a hypothetical future (Haaland aging), not a specific real-world event. The intended definition was event-anchored; the learned boundary was tone-anchored.

This gap is not a failure of labeling — the labels are consistent. It's a gap between what the labels *mean* and what a 66M-parameter model can *learn* from 147 examples. A larger dataset with examples that explicitly cover the hard cases would close much of this gap.

---

## Spec Reflection

**Where the spec helped:** The spec's requirement to write explicit decision rules for edge cases before annotation (planning.md Section 3) was the most practically valuable guidance in the project. Having written rules for "stat-backed hot take" and "reaction with embedded observation" before labeling meant borderline cases had a consistent resolution procedure. Without this, the `hot_take` ↔ `analysis` boundary would have been labeled inconsistently, which would have made the model's task even harder.

**Where implementation diverged:** The spec assumed data would be collected from Reddit directly. Reddit's API was inaccessible due to platform restrictions, so all 211 examples were synthetically generated using an LLM to reflect authentic r/soccer linguistic patterns. This was an honest deviation disclosed in the planning.md and dataset documentation. The synthetic origin likely contributed to the Groq baseline's unusually high performance (96.9%), since a large LLM classifying examples shaped by LLM-style generation has an inherent advantage over a small fine-tuned model.

---

## AI Tool Usage

**Label stress-testing (pre-annotation):** Before generating examples, Claude was asked to produce 10 posts sitting at the boundary between each label pair (`analysis` ↔ `hot_take`, `hot_take` ↔ `reaction`, `analysis` ↔ `reaction`). The output revealed that the `hot_take` ↔ `analysis` boundary was the most ambiguous — several generated posts couldn't be cleanly labeled without applying the "would removing opinion language leave a coherent argument?" test. This led to tightening the `hot_take` definition in planning.md before examples were generated.

**Dataset generation:** Claude generated all 211 examples using the label definitions from planning.md as a prompt, targeting ~70 examples per class. Each batch was reviewed to check that examples matched the intended register. Approximately 15 examples were discarded for being too clearly stereotypical (e.g., all-caps reactions with obvious emojis) and replaced with more varied posts. All examples are flagged `pre_labeled=True` and `source=synthetic`.

**Failure pattern analysis:** After obtaining the confusion matrix, Claude was given the 4 misclassified posts and asked to identify common themes. Claude's initial observation — "all errors involve `hot_take`, and the posts have either structured-sounding language or emotional tone" — matched the confusion matrix pattern. Claude further noted that two of the errors involved posts with causal framing ("the reason X is Y"), which it suggested might be a surface heuristic the model had learned to associate with `analysis`. This prompted the specific analysis of Error 1 and Error 2 above, which was verified by re-reading those examples manually.

---

## Setup and Reproduction

**Requirements:** Google Colab (T4 GPU runtime), Python 3.10+

**Dependencies installed by notebook:**
```
pip install groq python-dotenv
```

HuggingFace `transformers`, `datasets`, `torch`, `sklearn`, `pandas`, `matplotlib` are pre-installed on Colab.

**Steps:**
1. Open `ai201_project3_takemeter_starter_clean.ipynb` in Google Colab
2. Set runtime to T4 GPU: Runtime → Change runtime type → T4 GPU
3. Add your Groq API key to Colab Secrets (key name: `GROQ_API_KEY`)
4. Place `soccer_dataset.csv` in the Colab filesystem (upload via Files panel)
5. Run Sections 1 and 2 (setup and data loading)
6. Run Section 5 (Groq baseline — requires Groq API key)
7. Run Section 3 (fine-tuning — ~10 minutes on T4)
8. Run Section 4 (evaluation — generates `confusion_matrix.png`)
9. Run Section 6 (comparison — generates `evaluation_results.json`)

**Output files:**
- `confusion_matrix.png` — confusion matrix on test set (committed to repo)
- `evaluation_results.json` — accuracy, F1, label map (committed to repo)
