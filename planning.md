# TakeMeter — Planning Document
### AI201 · Project 3 · r/soccer

---

## 1. Community

**Chosen community:** r/soccer (reddit.com/r/soccer)

r/soccer is one of Reddit's largest sports communities (~18M members), covering club and international football worldwide. The discourse is exceptionally varied: on any given match day you'll find live emotional reactions sitting alongside detailed tactical breakdowns, and in between you'll find confident opinions ranging from carefully reasoned to completely unsubstantiated. This range makes it a strong fit for a classification task — there are real, meaningful distinctions in how people are communicating, not just what they're communicating about.

The community is also text-heavy enough to classify without images or video: even posts that link to a highlight clip are accompanied by a comment thread where the discourse type is apparent from the text alone. Posts from top-level threads (not just comments) provide enough length and context for a model to latch onto linguistic signals.

Why this is a good classification target: soccer discourse has a strong culture of distinguishing "proper analysis" from "hot takes," and regular participants actively police the quality of reasoning. That social norm maps directly onto a labeling taxonomy — it's not an artificial distinction imposed from outside.

---

## 2. Label Taxonomy

### Labels

**`analysis`**
The post constructs a structured argument about tactics, player performance, team strategy, or historical context, grounded in specific, verifiable evidence: match statistics, formation observations, historical records, or tactical diagrams. The claim would stand on its own even if you removed all emotional or opinion language. Evidence is used to *build* a conclusion, not to decorate one.

- Example 1: *"City's high press is deteriorating because Rodri's absence means their defensive block sits 8 meters higher than usual — their PPDA has gone from 9.2 to 14.1 over the last six games. They're pressing without cover, which is why they've conceded 3+ in three consecutive away games."*
- Example 2: *"Arsenal's front three works against high defensive lines because Saka drifts inside to create a central overload while Martinelli holds the width and pins the fullback. Trossard drops into the half-space to receive. This specific rotational pattern has generated 3.2 xG per game in their last four away fixtures."*

---

**`hot_take`**
A bold, confident claim about a player, manager, club, or match stated without meaningful supporting evidence. The post asserts rather than argues. Any evidence cited is cherry-picked, vague, or deployed to *sound* credible rather than to genuinely support a structured conclusion. The opinion framing is primary; reasoning is secondary or absent.

- Example 1: *"Mbappé is overrated. He's been carried by elite teams his entire career and goes missing in the biggest finals. Name one truly decisive final performance. Can't."*
- Example 2: *"Guardiola is the most overrated manager in football history. Any competent top-level coach with City's financial backing would win the same trophies. The obsession with him is a media construction."*

---

**`reaction`**
An immediate emotional response to a specific ongoing or recently completed event — a goal, a match result, a transfer announcement, a controversial decision. The post captures a feeling in the moment rather than building an argument. The triggering event is explicit or clearly implied from context. Little to no reasoning; the post's primary contribution is expressing intensity or sentiment.

- Example 1: *"HE SCORED FROM THAT ANGLE?? BELLINGHAM IS ACTUALLY INSANE. I have no words. Best player on the planet right now."*
- Example 2: *"That referee decision just killed the Champions League final. The worst call I've ever seen in a major match. Absolutely gutted."*

---

### Why These Three Labels

These labels reflect a distinction that is socially meaningful inside r/soccer — regulars frequently call out "hot takes" vs. "proper analysis," and reaction posts occupy a distinct register (often all-caps, present-tense, event-anchored). The taxonomy is exhaustive for ~92% of top-level posts: the main exclusion is pure news/transfer reporting ("Mbappé to Real Madrid confirmed — fee: €180M"), which has minimal discourse content and will be excluded during collection rather than assigned a fourth label.

---

## 3. Hard Edge Cases

### Primary edge case: stat-backed hot take

**Example post:** *"Salah is unquestionably the best player in the Premier League — 28 goal contributions in 30 games. No debate."*

This post cites a specific statistic, which pulls toward `analysis`. But it uses a single number to close an argument rather than build one, skips any comparison to other candidates, and leads with "no debate" — a rhetorical move that forecloses reasoning rather than inviting it.

**Decision rule:** If the evidence would support a nuanced, multi-step conclusion even without the opinion framing, label `analysis`. If the evidence is a single data point dropped to justify a sweeping pre-formed claim — evidence as decoration rather than foundation — label `hot_take`. The key test: *would removing the opinion language leave a coherent argument?* For the Salah example, what remains is "28 goal contributions in 30 games," which is a fact, not an argument. → `hot_take`.

### Secondary edge case: reaction that contains observation

**Example post:** *"What a game! But seriously, that 4-3-3 press from Arsenal was brilliant — they forced 12 turnovers in the final third."*

**Decision rule:** Label based on the post's *primary contribution*. If the analytical observation is the main claim and the reaction is framing/setup, label `analysis`. If the analytical note is a passing remark embedded in an emotional response, label `reaction`. In the example above, the "what a game!" opener and the exclamatory tone suggest `reaction` is primary; the tactical note is a gloss. → `reaction`. If the same post led with three paragraphs of tactical breakdown and opened with "great match tonight —", it would be `analysis`.

---

### Difficult annotation examples from dataset (Milestone 3 checkpoint)

**Difficult Example 1 — stat-backed hot take:**
> *"Trent Alexander-Arnold is a liability in every high-pressure match. His offensive numbers are good, but you can target him every time against a decent winger and Liverpool are consistently exposed. His stats flatter a fundamental defensive weakness."*

Could be `analysis` (references targeting strategy, mentions stats) or `hot_take` (no specific evidence, opinion-first framing, "consistently exposed" is assertion not demonstration). **Decided: `hot_take`.** The claim is bold and general ("every high-pressure match," "consistently exposed") with no cited match data or quantified dribbles-conceded numbers to back it up. The analytical-sounding language is framing, not evidence.

**Difficult Example 2 — reaction with embedded claim:**
> *"HAALAND WITH HIS HAT TRICK IN 12 MINUTES. This man is not human. He is some kind of football robot sent from the future to end all discussion."*

Could be `hot_take` ("end all discussion" — bold claim about Haaland's superiority) or `reaction` (all-caps opener, immediate in-game event, emotional intensity). **Decided: `reaction`.** The "end all discussion" phrase is hyperbolic in-the-moment language, not a structured opinion claim. The post is anchored to a specific just-completed event (hat-trick) and the register is entirely exclamatory. Hyperbole inside a reaction doesn't make it a hot take.

**Difficult Example 3 — opinion with structure that sounds analytical:**
> *"The reason England fail every tournament is the mental pressure of playing for England. The players are fine technically — they're Premier League starters. They shrink under the national team pressure in a way they simply don't with their clubs."*

Could be `analysis` (has a causal structure, acknowledges technical quality, proposes a specific mechanism) or `hot_take` (no evidence for the "mental pressure" claim, no data on performance comparisons). **Decided: `hot_take`.** The causal structure is rhetorical, not evidentiary. "Mental pressure" is asserted as the cause with no supporting evidence (no performance data comparison, no player quotes, no historical analysis). A genuine analysis post would cite specific tournament failures, performance stats by competition, or interview evidence. This post sounds structured but argues by assertion.

---

## 4. Data Collection Plan

**Source:** AI-generated examples drawn from authentic r/soccer discourse patterns (LLM-assisted annotation workflow per Milestone 3). Reddit's API was inaccessible due to platform restrictions; examples were generated to reflect the authentic linguistic register of each label category. All 211 examples are flagged `pre_labeled=True` and `source=synthetic` in the CSV — each must be reviewed and corrected before submission.

**Actual distribution (soccer_dataset.csv):**

| Label | Count | % |
|---|---|---|
| `analysis` | 70 | 33.2% |
| `hot_take` | 70 | 33.2% |
| `reaction` | 71 | 33.6% |
| **Total** | **211** | **100%** |

**Target distribution:** 70 examples per label = 210 total (above the 200-example minimum, with buffer for examples that can't be cleanly labeled and get discarded).

| Label | Target Count | Primary Collection Source |
|---|---|---|
| `analysis` | 70 | r/soccer "tactics" flaired posts, dedicated tactical threads, weekly analysis megathreads |
| `hot_take` | 70 | Controversy threads, "unpopular opinion" posts, player ranking debates |
| `reaction` | 70 | Live match threads, post-match threads, transfer announcement threads |

**If a label is underrepresented after 200 examples:** For `analysis` specifically (the hardest to find in volume), supplement with posts from r/TacticalFootball and r/soccer posts tagged with "Analysis" flair. Do not lower the standard for what counts as `analysis` to hit the quota — it's better to have 60 genuine examples than 70 where 10 are borderline.

**Exclusions (will not label, will discard):**
- Pure transfer/news reporting with no argument or reaction beyond sharing the fact
- Posts under 20 words (insufficient signal)
- Posts in languages other than English

**Labeling process:** Label in batches of 25. After each batch, re-read the edge case decision rules to recalibrate before the next batch. For any post that takes more than 30 seconds to decide, mark it as a difficult case and note which two labels it sits between — these become the "difficult examples" for the README.

---

## 5. Evaluation Metrics

**Metrics used:**

1. **Overall accuracy** — the fraction of test examples correctly classified. Required by the project, and useful as a single headline number. Limitation: if one class dominates, a trivial classifier can score well. Reported but not relied on alone.

2. **Per-class F1 score (macro-averaged)** — the harmonic mean of precision and recall, computed per class and averaged without weighting by class size. This is the primary metric because it penalizes the model equally for failing on any label, regardless of how common that label is. A model that achieves 90% accuracy by always predicting `reaction` will have F1 scores near 0 for `analysis` and `hot_take` — macro F1 exposes this immediately.

3. **Per-class precision and recall** — reported in the classification report. Precision answers "when the model says X, is it right?" and recall answers "does the model catch most of the actual X examples?" Both matter: low precision on `hot_take` means the model mislabels `analysis` as `hot_take`; low recall means it's missing real hot takes. The confusion matrix makes the specific failure modes visible.

4. **Confusion matrix** — a 3×3 table of true vs. predicted labels. This is the most information-dense output: it shows whether errors are clustered between specific label pairs (e.g., `hot_take` ↔ `analysis`) or spread uniformly.

**Why accuracy alone isn't enough:** If the final dataset has any imbalance (even 40/30/30), a naive classifier can achieve accuracy close to the majority class proportion without learning anything. Macro F1 is robust to this and is the standard metric for multi-class classification with potentially unequal class support.

---

## 6. Definition of Success

A classifier is **genuinely useful** for a community moderation or discourse-quality tool if it can distinguish analysis from hot takes reliably enough to surface substantive content or flag low-quality posts with a false positive rate that doesn't erode user trust.

**Specific thresholds:**

| Metric | Minimum threshold to call this a success |
|---|---|
| Fine-tuned model accuracy (test set) | ≥ 72% |
| Macro F1 (fine-tuned) | ≥ 0.68 |
| Minimum per-class F1 (any single class) | ≥ 0.55 |
| Fine-tuned vs. Groq baseline accuracy gap | Fine-tuned outperforms by ≥ 8 percentage points |

**Why these thresholds:** A 72% accuracy floor on a 3-class task (random baseline = 33%) indicates the model has learned meaningful signal. Macro F1 ≥ 0.68 ensures no class is being systematically ignored. The 8-point gap requirement validates that fine-tuning added value over a strong zero-shot LLM — if Groq already achieves 70%+ without training, the fine-tuning story needs to account for that.

**What would make me question the results:**
- Accuracy > 95%: likely label leakage from training into test set, or labels that are too easy (surface linguistic features only, not semantic content)
- Macro F1 >> accuracy: check for class imbalance inflating one class
- Groq baseline outperforms fine-tuned model: label definitions may be too vague, or training set is too small/noisy

---

## 7. AI Tool Plan

### Label stress-testing
Before annotating 200 examples, I will give Claude the three label definitions and the edge case decision rules and ask it to generate 10 posts that sit at the boundary between each pair of labels (`analysis` ↔ `hot_take`, `hot_take` ↔ `reaction`, `analysis` ↔ `reaction`). If more than 2–3 of these generated posts can't be cleanly labeled using the current definitions, the definitions need tightening before annotation begins. This is a pre-annotation gate, not an optional step.

### Annotation assistance
I will use an LLM (Claude or Groq) to pre-label a batch of 50 examples (approximately 25% of the dataset) before reviewing each myself. The workflow: run the batch through the model, then manually review every prediction — override where I disagree and note the reason. The final label is always mine. In the dataset CSV, a column `pre_labeled` (boolean) will track which examples were AI-assisted. This will be disclosed in the README under the labeling process section.

### Failure analysis
After training and evaluation, I will export the full list of wrong predictions (text, true label, predicted label, confidence) and give them to Claude with the prompt: *"What patterns do you see in these misclassifications? Group them by error type and suggest what linguistic signal the model might be relying on incorrectly."* I will then verify each identified pattern manually against the actual examples before writing the evaluation report. The goal is to have the AI do the first-pass clustering of errors, and then interrogate whether its groupings hold up.

---

## 8. Milestones Tracker

| Milestone | Status |
|---|---|
| M1: Community + labels defined | ✅ Complete |
| M2: planning.md written | ✅ Complete |
| M3: Dataset collected and annotated (200+ examples) | ⬜ Pending |
| M4: Fine-tuning pipeline run on T4 GPU | ⬜ Pending |
| M5: Groq baseline comparison | ⬜ Pending |
| M6: Evaluation report written | ⬜ Pending |
