# planning.md — TakeMeter: Goodreads Fantasy Review Classifier

---

## 1. Community

The community I am studying is the Goodreads fantasy review community, where readers share written reviews and ratings for fantasy books they have read. This community is a good fit for a classification task because the reviews do not always serve the same purpose. Some are quick expressions of approval or disapproval written more for the reviewer themselves, some are mostly plot retellings without much original thought, and some are in-depth evaluations of craft and themes that could meaningfully prompt discussion. This variety in purpose and quality makes the discourse rich enough to support a multi-label classification task.

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

### Summary
A review that primarily retells the plot or events of the book in sequence. The reviewer is describing what happens rather than evaluating or responding to it. There may be a brief opinion, but the dominant activity is narrating the book's content.

**Example 1:**
> "First, we meet Tane, who is preparing for a ceremony that will decide her future. Then a fugitive arrives and complicates everything. Her best friend Susa gets involved, and a chain of events follows that spans both the East and West storylines."

**Example 2:**
> "The story follows four main POVs across different regions. Each character has their own arc that slowly converges as the threat of the Nameless One grows. There are dragons, pirates, and a secret order of mages."

---

### Analysis
A review that evaluates the book by engaging with its craft, themes, character development, writing style, or by drawing comparisons to other works. The reviewer goes beyond describing what happened to arguing why it works or does not work. Emotion is welcome here, as long as it is supported by specific reasoning or evidence.

**Example 1:**
> "The world was complex and interesting, but since it is a standalone with four main POVs, it got overwhelming at times. The magic system was great and the plot was intriguing. What bothered me was that every character death felt sudden and detached, as if the authors needed casualties for the sake of the epic scale rather than letting those deaths carry weight. The battle at the end, which we wait for throughout the whole book, ended up feeling rushed and lukewarm."

**Example 2:**
> "Bardugo does something really interesting with power dynamics in this book. Compared to Shadow and Bone, the characters here have much more agency, and the heist structure forces them to rely on each other in ways that feel earned rather than convenient. The tension never lets up because the stakes feel personal, not just plot-driven."

---

## 3. Hard Edge Cases

### Edge Case 1: Summary with embedded opinions (Summary vs. Analysis)

Some reviews are structured like a plot walkthrough but include evaluative comments along the way, like "which I thought was unnecessary" or "this part dragged." The dominant activity is still describing events, not building an argument.

**Decision rule:** If the primary activity is walking through what happens in the book, even with scattered opinions mixed in, label it **summary**. Only label a review **analysis** if the evaluative argument is the dominant purpose of the review, not a side note inside a retelling.

**Example of this edge case:**
> The Priory of the Orange Tree review (organized by region: EAST, WEST, SOUTH, ELSEWHERE), which retells each storyline section by section with reactions and complaints woven throughout. Despite having strong opinions, the structure and dominant purpose is plot description. Labeled: **summary**.

---

### Edge Case 2: Emotional reviews with reasoning (Reaction vs. Analysis)

Some reviews use enthusiastic or emotional language ("I loved this!!!") but go on to explain specifically what made them feel that way, citing plot points, craft decisions, or comparisons.

**Decision rule:** If a review expresses strong emotion but supports that emotion with specific reasoning, craft observations, or plot points used as evidence, label it **analysis**. Only label a review **reaction** if the emotional response stands alone without substantive support.

---

## 4. Data Collection Plan

**Source:** UCSD Goodreads Dataset, Fantasy and Paranormal reviews subset.

**Target distribution:** Approximately 70 examples per label (210 total), to ensure no label exceeds 70% of the dataset and each label is meaningfully represented.

**Process:**
- Filter reviews to English only and set a minimum text length to avoid very short reviews that would almost always be reaction, which could create imbalance.
- Sample across star ratings (1 through 5) to capture a range of tones and purposes.
- After the first 100 labeled examples, check the distribution per label. If any label is below 15%, sample more deliberately from that type before continuing.

**If a label is underrepresented:** Summary reviews may be rarer than reactions in the dataset. If this happens, I will re-sample from shorter reviews or reviews that mention specific plot events in their opening lines, as these are more likely to be summary-style.

---

## 5. Evaluation Metrics

**Accuracy** gives an overall baseline, but it is not sufficient on its own because if the dataset has any imbalance (for example, analysis reviews being more common), a model that just predicts the majority class would still score reasonably high without learning anything useful.

The following metrics will also be used:

- **Per-class F1 score** for each of the three labels, because F1 accounts for both precision and recall and gives a clearer picture of how the model handles each label independently.
- **Precision for the analysis label specifically**, because the most harmful error in a real deployment would be labeling a reaction review as analysis. This would mean promoting a low-quality review as if it were substantive, which undermines the purpose of the classifier.
- **Confusion matrix**, to see exactly which labels are being confused with which. This is especially important for the summary/analysis boundary, which is the hardest distinction in this taxonomy.

---

## 6. Definition of Success

**Minimum viable performance:**
Overall accuracy of at least 75%, with precision on the analysis label above 80%. At this level, the classifier is reliably identifying substantive reviews without falsely promoting reactions, which is the most important error to avoid.

**Strong performance:**
Overall accuracy of 85% or above, with per-class F1 above 0.75 for all three labels. At this level, the classifier would be genuinely useful as a tool for surfacing review quality in a community context.

A result above 95% would be suspicious given the subjective nature of this task and should prompt a check for data leakage or labels that are too easy.

---

## 7. AI Tool Plan

### Label stress-testing
Before annotating 200 examples, use Claude to generate 8 to 10 reviews that sit at the boundary between two labels, particularly between summary and analysis, and between reaction and analysis. If any generated examples cannot be cleanly classified using the decision rules above, the definitions will be tightened before annotation begins.

### Annotation assistance
Given time constraints, Claude will be used to pre-label batches of reviews using the label definitions from this document. Every pre-labeled example will be manually reviewed and corrected before it enters the dataset. A notes column in the CSV will track which examples were pre-labeled and whether they were corrected. This will be disclosed in the AI usage section of the README.

### Failure analysis
After training and evaluation, the list of misclassified examples will be shared with Claude with a prompt asking it to identify any systematic patterns in the errors. Any patterns identified will be manually verified before being included in the evaluation report. The goal is to go beyond listing individual wrong predictions and identify what the model actually learned versus what it was intended to learn.