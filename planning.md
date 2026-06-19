# TakeMeter: Steam Review Discourse Quality Classifier
## Project Planning & Specification

---

## Community & Data Source
**Platform:** Steam User Reviews  
**Game:** Crimson Desert  
**Goal:** Build a fine-tuned DistilBERT classifier to evaluate the quality and type of discourse in player reviews.

---

## Label Taxonomy

We define three mutually exclusive categories for Steam review discourse:

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

## Hard Edge Cases & Resolution Rules

**Problem:** Some reviews mix categories. Example: *"This game is garbage, crashes every 5 mins lol 😂"*
- Is it a bug report (technical) or a joke (joke_meme)?

**Resolution Rule:**
- If the review **predominantly mentions a legitimate technical issue** (even with humor), classify as `technical`.
- If the review is **primarily humorous/joking with only incidental technical mention**, classify as `joke_meme`.
- When in genuine doubt, defer to the **strongest explicit signal**: "mostly this game sucks" alone = `opinion`, but "mostly this game sucks *because of X bug*" = `technical`.

**Other Edge Cases:**
- Very short reviews ("10/10" or "waste of money") → `opinion` (subjective judgment with no technical depth)
- Review mentions a bug but frames it as funny/meme → Classify by **primary intent**: if reporting the bug is the goal, `technical`; if the humor is the goal, `joke_meme`.

---

## Data Collection Plan

**Target:** Minimum 200 manually labeled Steam reviews for Crimson Desert.

**Collection Strategy:**
1. **Manual Review Crawl:** Hand-select reviews from Steam's review section, prioritizing recent and sorted by "helpful" votes to capture varied community voices.
2. **Sampling Approach:** Aim for balanced class distribution:
   - Target: ~30–35% `technical`, ~35–40% `opinion`, ~25–30% `joke_meme`
   - Hard constraint: **No single class exceeds 70%** of the total dataset
3. **Annotation Process:**
   - Manually label each review into one of the three categories
   - Record review text, assigned label, and optional notes (edge cases, reasoning, ambiguity flags)
   - Use independent spot-check (2-reviewer agreement on a random 10–20 sample) to verify consistency
4. **Storage:** All data stored in `dataset.csv` with columns: `text`, `label`, `notes`

**Rationale:** Manual curation ensures clean labels and allows us to document edge cases; balanced distribution prevents the model from learning a majority-class bias.

---

## Evaluation Metrics

We will report **Accuracy, Precision, Recall, and F1-score** for the test set.

**Why these metrics?**
- **Accuracy alone is insufficient** if class distribution is imbalanced. A naive classifier could achieve 70% accuracy by predicting the majority class everywhere.
- **Precision** (true positives / predicted positives) tells us: *"When the model says a review is `technical`, how often is it correct?"* Important for avoiding false positives.
- **Recall** (true positives / actual positives) tells us: *"Of all actual `technical` reviews, how many did the model catch?"* Important for avoiding false negatives and completeness.
- **F1-score** (harmonic mean of Precision & Recall) balances both concerns and is a standard single metric for classification performance.

**Per-class reporting:** We will compute Precision, Recall, and F1 **per label** to spot if the model struggles with a specific class (e.g., high recall on `opinion` but low recall on `technical`).

---

## Definition of Success (Deployment Threshold)

A model is ready for deployment if it achieves:
- **Overall Accuracy ≥ 80%** on the hold-out test set
- **Per-class F1-score ≥ 0.75** for all three labels
- **No single class with Recall < 0.70** (we cannot afford to miss 30% of any category)

**Rationale:**
- 80% accuracy is a reasonable human-baseline for this subjective task.
- F1 ≥ 0.75 ensures balanced precision/recall; no category is sacrificed.
- Minimum recall of 70% prevents any class from being systematically overlooked.

---

## AI Tool Plan

### 1. Label Stress-Testing
**Purpose:** Identify ambiguous reviews and validate label boundaries.
- Use an LLM (e.g., GPT-4 or Claude) to independently label a random 20-review sample from our dataset.
- Compare LLM predictions against our manual labels.
- Investigate mismatches: are they edge cases requiring rule refinement, or is our manual labeling inconsistent?
- Refine the `Hard Edge Cases` rules if needed.

### 2. Annotation Assistance
**Purpose:** Accelerate labeling of new reviews and catch human error.
- After training a baseline model, use it as a **pseudo-labeler** for unlabeled reviews.
- Use an LLM to review model predictions and flag low-confidence examples for human review.
- This speeds up dataset expansion while maintaining quality control.

### 3. Failure Analysis
**Purpose:** Understand why the model misclassifies reviews.
- Run inference on the test set; collect all false positives and false negatives.
- Use an LLM to analyze each failure:
  - Why is this review confusing? (ambiguous wording, sarcasm, mixed signals?)
  - Is the error due to model limitations or data labeling error?
  - What feature would have helped the model decide correctly?
- Document patterns in failures (e.g., "models struggle with sarcasm in opinion reviews").
- Use insights to improve data or prompt the fine-tuning strategy.

---

## Timeline & Next Steps
1. **Phase 1:** Collect and manually label 200+ reviews (target: 1 week)
2. **Phase 2:** Train DistilBERT model on labeled data (Colab notebook)
3. **Phase 3:** Evaluate on test set and run stress-testing with LLMs
4. **Phase 4:** Document results, error analysis, and reflection

---

*Document Version: 1.0*  
*Last Updated: 2026-06-19*
