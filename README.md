# TakeMeter: Steam Review Discourse Quality Classifier

## Project Overview

**TakeMeter** is a fine-tuned DistilBERT text classifier that evaluates the quality and type of discourse in Steam user reviews for *Crimson Desert*. The model categorizes reviews into three discourse types:

- **`technical`** — Reviews detailing game mechanics, optimization, bugs, hardware setup, or performance
- **`opinion`** — Subjective recommendations or critiques based on fun, value, or story (without technical depth)
- **`joke_meme`** — Humorous one-liners, chistes, or copypasta common in Steam review culture

This project demonstrates:
- Manual dataset curation from real Steam reviews
- Fine-tuning a pre-trained transformer model (DistilBERT)
- Evaluating model performance with multiple metrics (Accuracy, Precision, Recall, F1-score)
- AI-assisted failure analysis and label stress-testing

---

## Results Summary

### Overall Performance

| Metric | Value |
|--------|-------|
| Overall Accuracy | [INSERT: X%] |
| Dataset Size | [INSERT: # reviews] |
| Test Set Size | [INSERT: # reviews] |
| Model | DistilBERT (fine-tuned) |
| Training Framework | PyTorch + Hugging Face Transformers |

---

### Per-Class Metrics

| Label | Precision | Recall | F1-Score | Support |
|-------|-----------|--------|----------|---------|
| `technical` | [INSERT] | [INSERT] | [INSERT] | [INSERT] |
| `opinion` | [INSERT] | [INSERT] | [INSERT] | [INSERT] |
| `joke_meme` | [INSERT] | [INSERT] | [INSERT] | [INSERT] |

---

### Confusion Matrix

Below is the confusion matrix showing the model's prediction accuracy across all three classes:

```
[INSERT: Confusion Matrix Visualization or Image]
```

**Interpretation Notes:**
- [INSERT: Which classes does the model confuse most often?]
- [INSERT: Any patterns in false positives or false negatives?]

---

## Error Analysis

### Failure Case 1: [Brief Title]
**Original Review:**
```
[INSERT: Example review text]
```
**True Label:** `[INSERT: correct label]`  
**Predicted Label:** `[INSERT: model's prediction]`  
**Analysis:** [INSERT: Why did the model get this wrong? What was confusing about it?]

---

### Failure Case 2: [Brief Title]
**Original Review:**
```
[INSERT: Example review text]
```
**True Label:** `[INSERT: correct label]`  
**Predicted Label:** `[INSERT: model's prediction]`  
**Analysis:** [INSERT: Root cause analysis and observations]

---

### Failure Case 3: [Brief Title]
**Original Review:**
```
[INSERT: Example review text]
```
**True Label:** `[INSERT: correct label]`  
**Predicted Label:** `[INSERT: model's prediction]`  
**Analysis:** [INSERT: What feature or training strategy might have helped?]

---

## Key Insights

- **Strength:** [INSERT: What does the model do well?]
- **Weakness:** [INSERT: What struggles does it face?]
- **Surprise Finding:** [INSERT: Any unexpected behavior or pattern?]

---

## Reflection

### What Went Well
- [INSERT: Aspect of the project that succeeded]
- [INSERT: Positive learning or discovery]

### Challenges & Learnings
- [INSERT: Challenge encountered and how it was addressed]
- [INSERT: Key insight about the data, model, or evaluation process]

### Future Improvements
- [INSERT: How could the model or dataset be improved?]
- [INSERT: Additional strategies or approaches to try]

### Personal Takeaway
[INSERT: What did you learn about building text classifiers, working with Steam data, or fine-tuning transformers?]

---

## Repository Structure

```
ai201-project3-takemeter/
├── README.md                 # This file
├── planning.md               # Detailed project specification and strategy
├── dataset.csv               # Manually labeled Steam reviews (200+)
├── training_notebook.ipynb   # Google Colab notebook with DistilBERT fine-tuning
├── evaluation_results.json   # Model metrics and evaluation data
├── confusion_matrix.png      # Visualization of classification results
└── .gitignore
```

---

## How to Reproduce

### Prerequisites
- Python 3.8+
- PyTorch, Hugging Face Transformers, scikit-learn, pandas

### Steps
1. **Prepare Data:** Ensure `dataset.csv` is populated with labeled reviews (text, label, notes)
2. **Train Model:** Open `training_notebook.ipynb` in Google Colab and run all cells
3. **Evaluate:** Model automatically generates `evaluation_results.json` and `confusion_matrix.png`
4. **Review:** Check results in this README and analyze errors in the "Error Analysis" section

---

## References & Resources

- [DistilBERT Paper](https://arxiv.org/abs/1910.01108)
- [Hugging Face Transformers Documentation](https://huggingface.co/docs/transformers/)
- [Scikit-learn Metrics](https://scikit-learn.org/stable/modules/model_evaluation.html)

---

**Project Status:** [INSERT: In Progress / Complete]  
**Last Updated:** [INSERT: Date]  
**Author:** [INSERT: Your Name]
