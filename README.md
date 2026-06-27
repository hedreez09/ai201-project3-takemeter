# ai201-project3-takemeter
My text classifier project on AI Engineering journey

# TakeMeter: Student Study Comment Classifier

## Project Overview

TakeMeter is a fine-tuned text classification project that evaluates the type of discourse found in student study-related online discussions. The goal of this project is to classify student comments into meaningful categories based on the main purpose of the comment.

For this version of TakeMeter, I focused on student study discussions, where students often share study methods, complain about coursework, ask for motivation, or request learning resources. The classifier was designed to identify these patterns and assign each comment to one of four labels:

* `study_strategy`
* `complaint`
* `motivation_request`
* `resource_question`

The project compares a zero-shot Groq baseline model against a fine-tuned DistilBERT model trained on my labeled dataset.

---

## Community and Task

The chosen community is student study-related discussion spaces. These include the kinds of posts students make when discussing studying, assignments, exams, motivation, learning tools, and academic stress.

This community is a good fit for a classification task because student comments are varied but still follow recognisable patterns. Some posts give useful study advice, some express frustration, some ask for emotional support, and others ask for tools or resources. These categories are meaningful because they reflect different student needs.

The task is to classify each student's study comment into exactly one label.

---

## Labels

### `study_strategy`

A comment that gives a clear study method, learning technique, routine, or practical advice that could help someone study better.

Example:

> I started making flashcards after every lecture instead of waiting until exam week.

### `complaint`

A comment that mainly expresses frustration, stress, disappointment, or dissatisfaction about school, studying, exams, assignments, professors, or learning.

Example:

> I studied all weekend and the exam still looked nothing like what we practiced.

### `motivation_request`

A comment that asks for encouragement, discipline help, accountability, confidence, or support to keep studying.

Example:

> How do you make yourself start studying when you already feel behind?

### `resource_question`

A comment that asks for tools, apps, notes, videos, PDFs, books, websites, templates, practice questions, or other study resources.

Example:

> Does anyone know a free app that can turn notes into flashcards?

---

## Hard Edge Cases

Some comments could reasonably fit more than one label. I handled these cases by deciding the main purpose of the comment.

### Edge Case 1: Complaint vs. Motivation Request

Example:

> I failed my first quiz and feel discouraged. How do people bounce back?

This includes a negative experience, but the main purpose is asking for help and encouragement. I labeled this as `motivation_request`.

Decision rule: If the comment mainly asks for encouragement or help continuing, it should be labeled `motivation_request`. If it only expresses frustration without asking for support, it should be labeled `complaint`.

### Edge Case 2: Study Strategy vs. Resource Question

Example:

> I use flashcards for facts but practice problems for anything that requires solving.

This mentions flashcards and practice problems, but it is not asking for a tool or material. It is describing a study approach. I labeled this as `study_strategy`.

Decision rule: If the comment describes a method, label it `study_strategy`. If it asks where to find a tool, app, website, video, or material, label it `resource_question`.

### Edge Case 3: Motivation Request vs. Resource Question

Example:

> Does anyone have advice for studying after a long work shift?

This asks for advice, but the main need is motivation and routine support, not a specific app or study material. I labeled this as `motivation_request`.

Decision rule: If the student asks for emotional support, consistency, discipline, or motivation, label it `motivation_request`. If the student asks for a concrete tool, site, app, or study material, label it `resource_question`.

---

## Dataset

The dataset contains 200 labeled student study-style comments.

The CSV file has three columns:

* `text`
* `label`
* `notes`

The labels are balanced across the four categories:

| Label                | Count   |
| -------------------- | ------- |
| `study_strategy`     | 50      |
| `complaint`          | 50      |
| `motivation_request` | 50      |
| `resource_question`  | 50      |
| **Total**            | **200** |

The dataset was split into:

| Split          | Count |
| -------------- | ----- |
| Training set   | 140   |
| Validation set | 30    |
| Test set       | 30    |

The dataset used in this project contains student-study-style comments generated and reviewed for this project, modeled after common patterns in student study discussions. Each example was reviewed against the label definitions before training.

---

## Model and Training

The fine-tuned model used:

```text
distilbert-base-uncased
```

I used DistilBERT because it is a lightweight transformer model that is commonly used for text classification tasks. It is smaller and faster than BERT while still being strong enough for a small classification dataset.

The model was loaded with a classification head for four labels:

```python
LABEL_MAP = {
    "study_strategy": 0,
    "complaint": 1,
    "motivation_request": 2,
    "resource_question": 3
}
```

The notebook tokenized the dataset using the DistilBERT tokenizer and then fine-tuned the model on the training split.

### Hyperparameter Decision

The original training run used fewer epochs and performed poorly. The model showed low confidence predictions and frequently over-predicted `resource_question`.

To improve learning, I changed the number of training epochs to 5. After this change, the fine-tuned model learned the label boundaries much more clearly and reached 100% accuracy on the test set.

---

## Baseline Model

The baseline model was Groq's `llama-3.3-70b-versatile` used in a zero-shot setting.

This means the Groq model was not trained on my dataset. Instead, I gave it the label definitions and asked it to classify each test example using the prompt.

The baseline prompt included:

* the community
* the classification task
* each label definition
* one example per label
* an instruction to return only the label name

---

## Evaluation Results

Both models were evaluated on the same 30-example test set.

| Model                   | Accuracy |
| ----------------------- | -------- |
| Zero-shot Groq baseline | 96.67%   |
| Fine-tuned DistilBERT   | 100.00%  |

The fine-tuned DistilBERT model improved over the Groq baseline by 3.33 percentage points.

---

## Per-Class Metrics

### Fine-Tuned DistilBERT

The fine-tuned model correctly classified every test example.

| Label                | Precision | Recall | F1-score | Support |
| -------------------- | --------- | ------ | -------- | ------- |
| `study_strategy`     | 1.00      | 1.00   | 1.00     | 8       |
| `complaint`          | 1.00      | 1.00   | 1.00     | 7       |
| `motivation_request` | 1.00      | 1.00   | 1.00     | 8       |
| `resource_question`  | 1.00      | 1.00   | 1.00     | 7       |
| **Accuracy**         |           |        | **1.00** | **30**  |

### Zero-Shot Groq Baseline

The Groq baseline also performed strongly, but it made one mistake on the test set.

| Label                | Precision | Recall | F1-score | Support |
| -------------------- | --------- | ------ | -------- | ------- |
| `study_strategy`     | 0.89      | 1.00   | 0.94     | 8       |
| `complaint`          | 1.00      | 1.00   | 1.00     | 7       |
| `motivation_request` | 1.00      | 0.88   | 0.93     | 8       |
| `resource_question`  | 1.00      | 1.00   | 1.00     | 7       |
| **Accuracy**         |           |        | **0.97** | **30**  |

---

## Fine-Tuned Model Confusion Matrix

Rows represent the true labels. Columns represent the predicted labels.

| True Label           | Predicted `study_strategy` | Predicted `complaint` | Predicted `motivation_request` | Predicted `resource_question` |
| -------------------- | -------------------------- | --------------------- | ------------------------------ | ----------------------------- |
| `study_strategy`     | 8                          | 0                     | 0                              | 0                             |
| `complaint`          | 0                          | 7                     | 0                              | 0                             |
| `motivation_request` | 0                          | 0                     | 8                              | 0                             |
| `resource_question`  | 0                          | 0                     | 0                              | 7                             |

The confusion matrix shows that the fine-tuned model made no mistakes on the test set. Every prediction landed on the diagonal, meaning the predicted label matched the true label for all 30 examples.

---

## Sample Classifications

| Text                                                                                 | True Label           | Predicted Label      | Confidence | Why the Prediction Makes Sense                                                  |
| ------------------------------------------------------------------------------------ | -------------------- | -------------------- | ---------- | ------------------------------------------------------------------------------- |
| I split big chapters into smaller sections and test myself after each one.           | `study_strategy`     | `study_strategy`     | 0.99       | The comment describes a specific study technique.                               |
| This class rushes every topic and then acts surprised when people are confused.      | `complaint`          | `complaint`          | 0.99       | The comment expresses frustration about the class pace.                         |
| What helps you keep studying when your grades are not improving yet?                 | `motivation_request` | `motivation_request` | 0.99       | The comment asks for help staying motivated.                                    |
| Does anyone know a free app that can turn notes into flashcards?                     | `resource_question`  | `resource_question`  | 0.99       | The comment asks for a specific study tool.                                     |
| I use flashcards for facts but practice problems for anything that requires solving. | `study_strategy`     | `study_strategy`     | 0.99       | The comment explains how the student chooses study methods for different tasks. |

Note: Confidence values may vary slightly depending on the final notebook run. These examples represent the model behavior observed after the successful 5-epoch training run.

---

## Error Analysis and Failure Modes

The final 5-epoch fine-tuned model made no mistakes on the 30-example test set. However, an earlier training run performed poorly and revealed useful failure patterns.

In the earlier run, the model often over-predicted `resource_question` and produced low confidence scores around 0.25–0.29. Since this is close to random guessing for a four-class task, it suggested that the model had not learned strong decision boundaries yet.

| Text                                                                            | True Label           | Earlier Prediction  | Analysis                                                                                                                                                                                     |
| ------------------------------------------------------------------------------- | -------------------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| I split big chapters into smaller sections and test myself after each one.      | `study_strategy`     | `resource_question` | The earlier model confused study-method language with resource-seeking language. The word “chapters” may have made it associate the comment with study materials rather than strategy.       |
| This class rushes every topic and then acts surprised when people are confused. | `complaint`          | `resource_question` | The earlier model failed to identify frustration as the main signal. It may not have learned emotional complaint language strongly enough.                                                   |
| What helps you keep studying when your grades are not improving yet?            | `motivation_request` | `resource_question` | The earlier model treated a help-seeking motivation question like a resource request. This shows the boundary between asking for advice and asking for a specific resource can be difficult. |

The main error pattern in the weaker model was over-prediction of `resource_question`. This suggests that early training was not enough for the model to separate general help-seeking language from actual resource requests.

Changing the number of training epochs to 5 fixed this issue on the test set.

---

## Reflection: What the Model Learned vs. What I Intended

I intended the model to learn the main purpose of each comment. For example, a comment asking for a tool should be a `resource_question`, while a comment asking for encouragement should be a `motivation_request`.

After fine-tuning for 5 epochs, the model appeared to learn these distinctions clearly on the test set. The model correctly separated study methods, complaints, motivation requests, and resource questions.

However, the perfect test score should be interpreted carefully. A 100% accuracy score may mean that the label boundaries in this dataset were very clean. In messy real-world student communities, comments are often more ambiguous. A student might complain, ask for advice, and mention a resource all in the same post. Those mixed-intent comments would likely be harder for the model.

The model may have learned strong surface patterns in the dataset. For example:

* comments starting with “How do you...” often fit `motivation_request`
* comments asking “Does anyone know...” often fit `resource_question`
* comments using frustration words like “hate,” “tired,” or “burned out” often fit `complaint`
* comments describing routines or methods often fit `study_strategy`

These are useful patterns, but future work should test the model on more realistic, mixed, and ambiguous comments.

---

## What I Would Improve Next

If I continued this project, I would improve it in the following ways:

1. Add more real student discussion examples from public communities.
2. Include more ambiguous posts that mix complaint, motivation, and resource-seeking.
3. Add a second annotator to check label agreement.
4. Test the model on completely new posts outside the original dataset.
5. Build a simple interface where users can paste a study comment and receive a label with confidence.

---

## Spec Reflection

The planning document helped guide the project by forcing me to define my labels before training the model. This was useful because the labels controlled the whole project. Once the definitions were clear, it was easier to label examples consistently and write a strong Groq baseline prompt.

One way the implementation diverged from the original plan was the training configuration. The first fine-tuning run did not perform well, so I adjusted the number of training epochs to 5. This change was made because the earlier model had low confidence and many incorrect predictions. After increasing the epochs, the model learned the categories more effectively.

---

## AI Usage

I used AI assistance in several parts of this project.

### 1. Label Design and Planning

I used AI to help brainstorm the label taxonomy for student study discussions. The AI suggested labels such as `study_strategy`, `complaint`, `motivation_request`, and `resource_question`. I reviewed these labels and kept them because they were mutually exclusive enough for the dataset and meaningful for student study communities.

### 2. Dataset Drafting and Review

I used AI to help generate realistic student-study-style comments for the dataset. I reviewed the comments and labels against the label definitions before using them in the project. The dataset should be understood as a synthetic project dataset modeled after common patterns in student study discussions.

### 3. Debugging and Notebook Help

I used AI to debug notebook issues, including missing variables such as `LABEL_MAP` and `train_df`, Colab upload problems, and Groq prompt parsing errors. I reviewed the suggested fixes and applied the ones that matched the notebook errors.

### 4. Failure Analysis

I used AI to help identify patterns from the earlier failed training run. The main pattern identified was that the weaker model over-predicted `resource_question` and had low confidence around random guessing level. I verified this by reviewing the wrong predictions manually.

---

## Repository Structure

```text
ai201-project3-takemeter/
├── data/
│   └── takemeter_labeled_comments.csv
├── results/
│   ├── evaluation_results.json
│   └── confusion_matrix.png
├── Planning.md
└── README.md
```

---

## How to Run

1. Open the TakeMeter starter notebook in Google Colab.
2. Set the runtime to T4 GPU.
3. Upload `data/takemeter_labeled_comments.csv`.
4. Define the label map:

```python
LABEL_MAP = {
    "study_strategy": 0,
    "complaint": 1,
    "motivation_request": 2,
    "resource_question": 3
}
```

5. Run the dataset loading and validation cells.
6. Run the train/validation/test split.
7. Run tokenization.
8. Run the Groq baseline.
9. Fine-tune DistilBERT.
10. Evaluate the model and export the results.

---

## Demo Video

Demo video link:

```text
PASTE_DEMO_VIDEO_LINK_HERE
```

In the demo, I show several student study comments being classified by the fine-tuned model, explain one correct prediction, discuss an earlier failure case, and walk through the evaluation results.
