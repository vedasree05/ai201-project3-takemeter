# planning.md — TakeMeter: Goodreads Fantasy Review Classifier

---

## 1. Community

The community I am studying is the Goodreads fantasy review community, where readers share written reviews and ratings for fantasy books they have read. This community is a good fit for a classification task because the reviews do not always serve the same purpose. Some are quick expressions of approval or disapproval written more for the reviewer themselves, and some are in-depth evaluations of craft and themes that could meaningfully prompt discussion. This variety in purpose and quality makes the discourse rich enough to support a classification task.

Data is sourced from the **UCSD Goodreads Dataset (Fantasy and Paranormal subset)**, a publicly available academic dataset of scraped Goodreads reviews collected in 2017.

---

## 2. Labels and Examples

### Reaction
A review that expresses an emotional response to the book with little elaboration. The reviewer is focused on how the book made them feel, not on reasoning through why.

**Example 1:**
> "One of the best books I've ever read. Rich in everything you could want and need in a fantasy. Please don't sleep on this book. Just don't. Read it."

**Example 2:**
> "I am absolutely devastated. I ugly cried for an hour. Five stars, no notes."

---

### Analysis
A review that evaluates the book by engaging with its craft, themes, character development, writing style, or by drawing comparisons to other works. The reviewer goes beyond describing what happened to arguing why it works or does not work. Emotion is welcome here, as long as it is supported by specific reasoning or evidence.

**Example 1:**
> "The world was complex and interesting, but since it is a standalone with four main POVs, it got overwhelming at times. The magic system was great and the plot was intriguing. What bothered me was that every character death felt sudden and detached, as if the authors needed casualties for the sake of the epic scale rather than letting those deaths carry weight. The battle at the end, which we wait for throughout the whole book, ended up feeling rushed and lukewarm."

**Example 2:**
> "Bardugo does something really interesting with power dynamics in this book. Compared to Shadow and Bone, the characters here have much more agency, and the heist structure forces them to rely on each other in ways that feel earned rather than convenient. The tension never lets up because the stakes feel personal, not just plot-driven."

---

## 3. Hard Edge Cases

### Edge Case: Emotional reviews with reasoning (Reaction vs. Analysis)

Some reviews use enthusiastic or emotional language ("I loved this!!!") but go on to explain specifically what made them feel that way, citing plot points, craft decisions, or comparisons to other works.

**Decision rule:** If a review expresses strong emotion but supports that emotion with specific reasoning, craft observations, or plot points used as evidence, label it **analysis**. Only label a review **reaction** if the emotional response stands alone without substantive support.

**Note on taxonomy change:** A summary label was initially planned to capture reviews that primarily retell plot events. After sampling 300 reviews from the dataset, summary reviews were found to represent only about 10-12% of the data, making it impossible to collect a balanced number of examples without artificially skewing the sample. The taxonomy was narrowed to two labels to reflect what the data actually contains and to produce a more reliable classifier.

---

## 4. Data Collection Plan

**Source:** UCSD Goodreads Dataset, Fantasy and Paranormal reviews subset.

**Target distribution:** Approximately 150 examples per label (300 total), to ensure neither label exceeds 70% of the dataset and both are meaningfully represented.

**Process:**
- Set a minimum text length of 150 characters to avoid very short reviews that would almost always be reaction and skew the distribution.
- Sample across star ratings to capture a range of tones and purposes.
- Use LLM pre-labeling on batches of 50, followed by manual review and correction of every example before it enters the final dataset.

**Observed distribution:** After pre-labeling 300 reviews, analysis accounts for roughly 60% of examples and reaction for roughly 40%, which is within an acceptable range for training.

---

## 5. Evaluation Metrics

**Accuracy** gives an overall baseline, but it is not sufficient on its own because if the dataset has any imbalance, a model that just predicts the majority class would still score reasonably high without learning anything useful.

The following metrics will also be used:

- **Per-class F1 score** for both labels, because F1 accounts for both precision and recall and gives a clearer picture of how the model handles each label independently.
- **Precision for the analysis label specifically**, because the most harmful error in a real deployment would be labeling a reaction review as analysis. This would mean promoting a low-quality review as if it were substantive, which undermines the purpose of the classifier.
- **Confusion matrix**, to see exactly which labels are being confused with which and whether errors are symmetric or skewed in one direction.

---

## 6. Definition of Success

**Minimum viable performance:**
Overall accuracy of at least 75%, with precision on the analysis label above 80%. At this level, the classifier is reliably identifying substantive reviews without falsely promoting reactions, which is the most important error to avoid.

**Strong performance:**
Overall accuracy of 85% or above, with per-class F1 above 0.75 for both labels. At this level, the classifier would be genuinely useful as a tool for surfacing review quality in a community context.

A result above 95% would be suspicious given the subjective nature of this task and should prompt a check for data leakage or labels that are too easy.

---

## 7. AI Tool Plan

### Label stress-testing
Before finalizing the taxonomy, Claude was used to identify boundary cases between labels. The stress-testing process revealed that the summary label was too rare in real data to be viable, which led to the decision to narrow the taxonomy to two labels.

### Annotation assistance
Claude was used to pre-label batches of 50 reviews at a time using the label definitions from this document. Every pre-labeled example was manually reviewed and corrected before entering the dataset. A notes column in the CSV tracks which examples were pre-labeled and whether they were corrected. This is disclosed here as part of the AI usage record.

### Failure analysis
After training and evaluation, the list of misclassified examples will be shared with Claude with a prompt asking it to identify any systematic patterns in the errors. Any patterns identified will be manually verified before being included in the evaluation report. The goal is to go beyond listing individual wrong predictions and identify what the model actually learned versus what it was intended to learn.