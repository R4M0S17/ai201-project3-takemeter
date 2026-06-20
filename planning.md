# TakeMeter: Steam Review Discourse Quality Classifier

## Project Planning & Specification

_A comprehensive specification for building and evaluating a fine-tuned transformer model on Steam user review discourse._

---

## 1. Community & Data Source

### Platform & Domain

**Primary Platform:** Steam Community (Steam User Reviews)  
**Selected Game:** Crimson Desert  
**Rationale:** Steam reviews represent a diverse discourse landscape. Reviews range from deep engineering/technical breakdowns discussing game mechanics, optimization, and physics systems to casual joke-and-meme commentary common in Steam culture. This natural variance across three distinct discourse types makes the corpus an excellent fit for developing a multi-class discourse quality classifier.

### Project Goal

Build a fine-tuned DistilBERT transformer-based classifier that accurately predicts the discourse type of player reviews across three mutually exclusive categories: technical, opinion, and joke_meme. The classifier will enable platform moderators and developers to quickly identify technical bug reports, genuine user feedback, and community humor in large review volumes.

---

## 2. Label Taxonomy

We define three mutually exclusive, non-overlapping categories for classifying Steam review discourse. The boundaries between categories are sharp and enforced through explicit edge-case resolution rules (see Section 3).

### Motivation for Three-Class Design

- **`technical`**: Enables developers and moderators to prioritize actionable bug reports and system requirements information.
- **`opinion`**: Captures subjective user feedback for understanding player sentiment and game value perception.
- **`joke_meme`**: Identifies community culture and entertainment value; important for toxicity filtering and engagement moderation.

### Category Definitions

### 1. `technical`

**Definition:** Reviews that focus on game mechanics, optimization, technical setup/hardware requirements, bugs, performance, or physics systems. These reviews provide specific technical information or bug reports.

**Example 1:**

```
"Game crashes on RTX 3080 with high-res textures enabled. Happens consistently
at the capital city loading screen. Reverting to driver 531.79 fixed it for me.
Devs should look into DX12 memory leaks."
```

**Example 2:**

```
"The physics engine has a clipping bug where players fall through geometry
on the bridge near the fort. Also, the frame rate tanks from 120 to 45 fps
during castle sieges. Performance is tied to draw distance—reducing it helps."
```

---

### 2. `opinion`

**Definition:** Standard recommendations or critiques based on subjective assessment of fun, value, story, character, or overall enjoyment. These reviews lack deep technical breakdown and express personal preference without mechanics/bug focus.

**Example 1:**

```
"Amazing story and gorgeous world, but the pacing feels slow in Act 2.
Definitely worth $39.99 if you enjoy open-world RPGs. Not for everyone,
but I got 80 hours of fun out of it."
```

**Example 2:**

```
"The combat is satisfying and the side quests are well-written. However,
the main character feels one-dimensional. I'd recommend it for fantasy fans,
but maybe wait for a 20% discount if you're on the fence."
```

---

### 3. `joke_meme`

**Definition:** Humorous one-liners, chistes, copypasta, or meme-style comments common in Steam review culture. These are not sincere critiques but entertainment/community references.

**Example 1:**

```
"10/10 best game ever, would get scammed again ✓"
```

**Example 2:**

```
"A surprise to be sure, but a welcome one. Prequel memes aside,
graphics are bussin no cap fr fr"
```

---

## 3. Hard Edge Cases & Resolution Rules

### Primary Decision Framework

**Central Principle:** Ambiguous reviews are resolved by identifying the **primary communicative intent**—the review's dominant purpose, not peripheral details.

### Canonical Boundary Cases

**Case 1: Technical Issue Presented Humorously**  
_Example: "This game crashes every 5 minutes lol 😂 RTX 4090 and latest drivers. Unplayable."_

**Decision Rule:**  
If the review **explicitly documents a reproducible technical issue with specifics** (hardware, driver version, reproducible trigger), classify as **`technical`**, even if humor is present. The primary intent is bug reporting.

---

**Case 2: Humor That Mentions a Technical Detail Incidentally**  
_Example: "10/10 would buy again, even though my GPU melts into the keyboard ✓"_

**Decision Rule:**  
If the review is **primarily humorous with only incidental technical mentions** (no specifics, no reproducibility info), classify as **`joke_meme`**. The humor is the communication goal; the technical mention is a vehicle for the joke.

---

**Case 3: Opinion Masked as Technical Concern**  
_Example: "The game has terrible optimization. My old laptop can't run it. Not worth the price."_

**Decision Rule:**  
If the review **lacks specific technical troubleshooting, driver versions, or reproducibility details**, but expresses general complaints tied to personal circumstances, classify as **`opinion`**. The reviewer is saying the game doesn't fit their needs, not reporting a bug.

---

### Secondary Edge Cases

| Scenario                                                         | Label       | Rationale                                                                  |
| ---------------------------------------------------------------- | ----------- | -------------------------------------------------------------------------- |
| Very short ("10/10", "waste of money", "amazing")                | `opinion`   | Subjective judgment with no technical depth or structured humor.           |
| Bug mentioned + framed as meme + no reproducibility details      | `joke_meme` | The joke is the primary artifact; bug mention is flavor.                   |
| Detailed bug report with joke formatting (all caps, emoji, etc.) | `technical` | Primary intent is to report the bug; style is secondary.                   |
| "Game bad" without elaboration                                   | `opinion`   | No technical specifics; pure subjective critique.                          |
| Reference to game lore, meme, or copypasta                       | `joke_meme` | Community culture and entertainment; not a technical or opinion statement. |

---

## 4. Data Collection & Annotation

### Collection Strategy

**Manual Curation Approach:** Reviews were hand-selected from Steam's review section for Crimson Desert, prioritizing a mix of recent submissions and high-helpful-vote entries to capture the breadth of community discourse. This manual approach ensures clean labels and enables systematic documentation of edge cases and borderline classifications.

**Data Extraction Process:**  
Raw review text blocks were extracted from the Steam web interface and compiled into a temporary buffer. LLM-assisted parsing (detailed in Section 7) was then applied to structure and validate the entries before final manual review.

---

### Annotated Dataset: Current Status

**Total Dataset Size:** 222 reviews (223 lines including CSV header)

**Class Distribution:**

| Class       | Count   | Percentage |
| ----------- | ------- | ---------- |
| `opinion`   | 132     | 59.5%      |
| `technical` | 42      | 18.9%      |
| `joke_meme` | 49      | 22.1%      |
| **Total**   | **222** | **100.0%** |

**Distribution Analysis:**

- No single class exceeds the 70% threshold, ensuring the dataset is balanced enough to prevent trivial majority-class learning.
- `opinion` is the largest class (59.5%), reflecting the prevalence of subjective feedback in player reviews.
- `technical` (18.9%) is the smallest class, requiring careful model evaluation to ensure the model does not systematically ignore bug reports.
- `joke_meme` (22.1%) is well-represented, indicating active community participation in humor and meme culture.

---

### Quality Assurance

**Annotation Consistency:**  
All 222 reviews were manually labeled by the primary annotator using the taxonomy and edge-case rules defined in Section 3. Spot-check validation on a random sample of 15–20 reviews was performed to verify label consistency and rule application.

**Storage Format:**  
All data is stored in `dataset.csv` with the following columns:

- `text`: The full review text
- `label`: One of `technical`, `opinion`, or `joke_meme`
- `notes`: Optional documentation of edge cases, ambiguity flags, or reasoning for borderline classifications

**Rationale for Manual Curation:**  
Manual review ensures clean, high-confidence labels and provides an audit trail for edge cases. This is particularly important for validating the model's learned boundaries later during failure analysis (Section 7).

---

## 5. Evaluation Metrics & Why Accuracy Alone Is Insufficient

### Primary Metrics

We will report the following metrics on the hold-out test set:

1. **Overall Accuracy:** (True Positives + True Negatives) / Total Predictions
2. **Per-Class Precision, Recall, and F1-Score** (computed for each label independently)
3. **Macro-Averaged Metrics:** Unweighted average across all three classes (prevents bias toward the majority class)

---

### Why Accuracy Alone Is Misleading

Given the dataset distribution:

- `opinion` ≈ 59.5% (majority class)
- `technical` ≈ 18.9% (smallest class)
- `joke_meme` ≈ 22.1% (minority class)

**A naive "lazy" classifier that predicts `opinion` for every review would achieve approximately 59.5% accuracy without learning any meaningful boundaries.** While accuracy would be above 50%, such a model would completely fail to identify technical bug reports (18.9% recall on `technical` ≈ 0%), making it useless in practice.

This phenomenon is known as **class imbalance bias**. Accuracy rewards majority-class dominance; it cannot signal whether the model has learned to distinguish between minority classes.

---

### Per-Class Metrics Interpretation

**Precision (Positive Predictive Value)**  
_Formula:_ TP / (TP + FP)  
_Interpretation:_ "When the model predicts a review is `X`, how often is it correct?"

- High precision on `technical` means: when the model flags a review as a bug report, we can trust it with high confidence.
- High precision on `joke_meme` means: false alarms for community humor are rare.

**Recall (Sensitivity / True Positive Rate)**  
_Formula:_ TP / (TP + FN)  
_Interpretation:_ "Of all actual reviews of type `X`, how many did the model catch?"

- High recall on `technical` means: we are not missing critical bug reports (few false negatives).
- High recall on `opinion` means: we capture the breadth of genuine user feedback.

**F1-Score**  
_Formula:_ 2 × (Precision × Recall) / (Precision + Recall)  
_Interpretation:_ Harmonic mean of precision and recall; balances avoiding false positives and false negatives.

- F1 is insensitive to class imbalance and provides a single, interpretable metric for each class.

---

### Reporting Standard

We will compute and report Precision, Recall, and F1-score **per label** in a confusion matrix and per-class metric table. This allows us to identify if the model struggles with a specific class (e.g., high recall on `opinion` but low recall on `technical`) and to trace performance asymmetries back to data quality or class-specific linguistic patterns.

---

## 6. Definition of Success & Deployment Threshold

### Success Criteria

A fine-tuned DistilBERT model is considered production-ready if it achieves:

1. **Baseline Beating:** Outperforms the Llama 3.3 zero-shot baseline by **at least 10 percentage points in overall accuracy**
2. **Per-Class F1-Score:** **F1-score ≥ 0.70** for all three classes (`technical`, `opinion`, `joke_meme`)
3. **Recall Guarantee:** **No class with Recall < 0.65** (we cannot afford to miss more than 35% of any category, especially technical bug reports)
4. **Minimum Overall Accuracy:** **≥ 75%** on the hold-out test set

### Rationale

**Baseline Comparison (Llama 3.3 Zero-Shot):**  
A large language model with no fine-tuning provides an upper-bound baseline for "zero-shot performance." Since our domain and labels are specialized, we expect our fine-tuned model to substantially exceed zero-shot performance. A 10-percentage-point improvement is a meaningful, demonstrable gain showing that fine-tuning is effective.

**F1-Score ≥ 0.70 Across All Classes:**

- Ensures balanced precision and recall; neither false positives nor false negatives dominate.
- 0.70 F1 is a standard industry threshold for acceptable multi-class classification.
- Particularly important for the minority class (`technical`, 18.9%): an F1 below 0.70 would indicate the model is unreliable at identifying bug reports.

**Recall ≥ 0.65 (Especially for `technical`):**

- Prioritizes **completeness**: we must catch the majority of actual technical issues, even if it means some false positives (low precision is worse than low recall for safety).
- A 65% recall threshold means missing no more than 35% of issues in any category.
- For `technical`: 65% recall means at least 2 out of every 3 bug reports are flagged, providing useful signal to developers.

**Overall Accuracy ≥ 75%:**

- 75% accuracy is a reasonable human-expert baseline for this nuanced discourse classification task.
- Above 75%, the model provides practical value; below 75%, the error rate is too high for confident deployment.

---

## 7. AI Tool Integration Plan

LLMs are used strategically at three critical stages: **data preparation, label validation, and failure analysis**. This approach balances automation with human oversight.

---

### 7.1 Label Stress-Testing (Boundary Case Generation & Rule Validation)

**Purpose:** Systematically test the boundaries of our taxonomy by generating synthetic edge cases and comparing LLM classifications against human judgment, thereby refining rules.

**Execution:**

1. **Synthetic Edge Case Generation:**
   - Prompt Claude or Sonnet to generate 15–20 **synthetic but realistic** Steam reviews that occupy each boundary:
     - A technical review written in humorous style (to test `technical` ↔ `joke_meme` boundary)
     - An opinion review that mentions a bug incidentally (to test `opinion` ↔ `technical` boundary)
     - A joke that references technical jargon but has no actionable information (to test `joke_meme` ↔ `technical` boundary)
2. **Independent LLM Classification:**
   - Provide the generated reviews to a second LLM instance (or a different model: GPT-4, Llama, etc.) without the annotator's labels.
   - Request classification into our three categories with explanation.
3. **Mismatch Analysis:**
   - Compare LLM predictions against the intended label (as defined by the prompt).
   - Where mismatches occur, investigate:
     - Does the LLM's interpretation reveal ambiguity in our rules?
     - Does our rule set need clarification or an additional sub-rule?
     - Is the synthetic example actually ambiguous, or is the LLM genuinely wrong?
4. **Rule Refinement:**
   - Update Section 3 (`Hard Edge Cases & Resolution Rules`) with new decision thresholds or examples.
   - Document the rationale for each rule update.

**Expected Outcome:** A more robust, LLM-validated taxonomy that captures linguistic edge cases humans might miss.

---

### 7.2 Annotation Assistance (LLM-Powered Parsing & Cleaning)

**Purpose:** Accelerate structured annotation by using LLMs to parse unstructured text and flag quality issues, while maintaining human review as the final arbiter.

**Execution:**

1. **Dirty Text Ingestion:**
   - Raw review text from Steam is often copy-pasted with formatting artifacts: line breaks, HTML entities, extra whitespace, Unicode rendering issues, etc.
   - These reviews are compiled into a temporary buffer.

2. **Quality Metrics:**
   - Document the cleaning error rate (% of rows requiring human correction).
   - If error rate > 5%, rerun Sonnet with refined prompts or switch to manual-only mode for remaining batches.

3. **Dataset Expansion:**
   - Once baseline model is trained, use its predictions on newly labeled reviews as a **pseudo-label confidence signal**.
   - Reviews where the model and human agree are added to the labeled pool; disagreements are escalated for review.
   - This allows incremental dataset expansion while controlling for annotation drift.

**Benefits:**

- Reduces manual parsing time by ~70% (Sonnet handles formatting, human validates meaning).
- Maintains label quality: humans remain the source-of-truth annotator.
- Provides an audit trail (original vs. cleaned) for reproducibility.

---

### 7.3 Failure Analysis (Systematic Error Pattern Detection)

**Purpose:** Understand why the fine-tuned model misclassifies reviews; identify systemic weaknesses and use insights to improve training or data labeling.

**Execution:**

1. **Error Collection:**
   - Run inference on the hold-out test set.
   - Extract all false positives and false negatives; group by:
     - Predicted class
     - True class
     - Confidence score (if available)
   - Example groups: _False Negatives (True: `technical`, Pred: `opinion`)_, _False Positives (True: `joke_meme`, Pred: `opinion`)_, etc.

2. **LLM Error Analysis:**
   - For each group, pass 3–5 representative examples to an LLM with the following prompt:

     ```
     The model misclassified the following reviews. For each, hypothesize why:
     1. What linguistic signals did the model likely miss?
     2. Is the error due to model confusion or human labeling error?
     3. What textual pattern (sarcasm, rhetorical questions, mixed signals) might explain the error?
     4. What additional feature or rule would have helped?

     Review 1 (Predicted: opinion | True: technical):
     [review text]

     [Continue for other reviews...]
     ```

   - The LLM outputs hypotheses for each error.

3. **Pattern Synthesis:**
   - Aggregate LLM hypotheses to identify **systemic patterns**:
     - _"The model struggles with sarcasm in opinion reviews; it interprets 'This game is great... for wasting $40' as genuinely positive."_
     - _"Short, 2–3 word reviews are systematically mislabeled as `joke_meme` even when they are opinion."_
     - _"Technical reviews without explicit keywords (bug, crash, lag) are misclassified as opinion."_
   - Rank patterns by frequency and impact.

4. **Root Cause Investigation:**
   - For top patterns, determine root cause:
     - **Model limitation:** The transformer lacks contextual understanding of sarcasm or negation.
     - **Data imbalance:** The training set has few examples of sarcastic opinion reviews.
     - **Labeling error:** The annotator was inconsistent in labeling short reviews; they should be re-reviewed.
     - **Ambiguous rule:** The edge-case rule in Section 3 is unclear or needs refinement.

5. **Recommendations & Iteration:**
   - Document recommended actions:
     - _"Augment training data with 20 synthetic sarcastic opinion examples."_
     - _"Retrain with class weights: `opinion` = 1.0, `technical` = 2.0, `joke_meme` = 1.5."_
     - _"Re-label the 12 short reviews (< 5 words) using updated rules and recheck consistency."_
   - Implement highest-impact recommendations and retrain.

**Expected Outcome:** Structured understanding of model failure modes; data-driven prioritization of improvements.

---

### 7.4 Integration Timeline

| Phase             | Task                                                 | Tool                          | Outcome                                              |
| ----------------- | ---------------------------------------------------- | ----------------------------- | ---------------------------------------------------- |
| **Pre-Training**  | Generate 20 synthetic boundary-case reviews          | Claude (creative generation)  | Validated taxonomy rules (Section 3)                 |
| **Data Prep**     | Clean 222+ raw review texts                          | Claude Sonnet (batch parsing) | Structured CSV, < 5% error rate                      |
| **Post-Training** | Analyze 15–30 representative failures                | Claude (pattern recognition)  | Failure pattern report & improvement recommendations |
| **Iteration**     | Implement top recommendations (data aug, retraining) | Manual + fine-tuning          | Improved model checkpoint                            |

---

## 8. Implementation Timeline & Milestones

| Phase                                   | Objectives                                                                                                                                         | Status         | Notes                                                 |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- | ----------------------------------------------------- |
| **Phase 1: Data Preparation**           | • Collect 200+ Steam reviews • Apply LLM-assisted parsing & cleaning • Manually label and validate dataset                                         | ✅ Complete    | 222 reviews collected and annotated                   |
| **Phase 2: Label Validation**           | • Generate 15–20 synthetic boundary-case reviews • Conduct LLM stress-testing of taxonomy • Refine edge-case rules if needed                       | 🔄 In Progress | Planned before model training begins                  |
| **Phase 3: Model Development**          | • Split data into train (70%) / validation (15%) / test (15%) • Fine-tune DistilBERT on labeled reviews • Hyperparameter tuning and early stopping | ⏳ Pending     | Colab notebook environment                            |
| **Phase 4: Evaluation & Analysis**      | • Report Accuracy, Precision, Recall, F1 per class • Compare against Llama 3.3 zero-shot baseline • Conduct systematic failure analysis            | ⏳ Pending     | Expected output: confusion matrix + per-class metrics |
| **Phase 5: Iteration & Refinement**     | • Implement top failure-analysis recommendations • Retrain model with improvements • Final validation against success criteria                     | ⏳ Pending     | Multiple iterations possible                          |
| **Phase 6: Documentation & Reflection** | • Write comprehensive final report • Document insights, surprises, and limitations • Prepare presentation/demo                                     | ⏳ Pending     | Milestone 2 deliverable                               |

---

## 9. Success Metrics Summary

**Model Performance Targets (Section 6):**

- Overall Accuracy: ≥ 75%
- Per-Class F1-Score: ≥ 0.70 (all three classes)
- Per-Class Recall: ≥ 0.65 (all three classes)
- Baseline Beating: ≥ 10% absolute improvement over Llama 3.3 zero-shot

**Data Quality Targets (Section 4):**

- Dataset size: 222 reviews (✅ achieved)
- Class balance: No class > 70% (✅ achieved: max is 59.5%)
- Annotation consistency: < 5% label discrepancies on spot-check (target)

**Process Quality Targets:**

- Taxonomy rule clarity: LLM stress-testing detects no ambiguities (target)
- Annotation parsing accuracy: < 5% error rate on Claude Sonnet cleaning (target)

---

_Document Version: 2.0_  
_Last Updated: 2026-06-19_  
_Status: Milestone 2 Planning Complete_
