# Evaluation Metrics

## Classification Metrics

For classification scenarios, evaluation metrics are different ways of looking at how well a model's predictions match the true labels.

### Confusion Matrix

Confusion Matrix is a grid supposed to map the predict values with the actual values.
Here we use the example of a binary classification matrix to understand different kinds of metrics.

|                    | Actual positive    | Actual negative    |
| -------------------| ------------------ | ------------------ |
| Predicted positive | True Positive (TP) | False Positive (FP)|
| Predicted negative | False Negative (FN)| True Negative (TN) |

- True Positive (TP): the model correctly predicts the positive class.
- True Negative (TN): the model correctly predicts the negative class.
- False Positive (FP): the model predicts positive when the actual class is negative. This is also called a **Type I error** or false alarm.
- False Negative (FN): the model predicts negative when the actual class is positive. This is also called a **Type II error** or missed detection.

### Accuracy

Accuracy is the proportion of correct predictions.

$$
\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}
$$

Accuracy can be misleading for **imbalanced datasets**. For example: a model that always predicts the majority class can have high accuracy while completely failing to detect the minority class.

### Precision

Precision is the proportion of positive predictions that were correct.

$$
\text{Precision} = \frac{TP}{TP + FP}
$$

High precision means the model produces few false positives. It is especially important when acting on a false alarm is costly, such as incorrectly marking legitimate email as spam.

### Recall

Recall (sensitivity), also called the **true positive rate (TPR)** is the proportion of actual positives that were correctly identified by the model detect.

$$
\text{Recall} = \frac{TP}{TP + FN}
$$

High recall means the model produces few false negatives. It is especially important when missing a positive case is costly, such as failing to detect a disease.

### Specificity

Specificity, also called the **true negative rate (TNR)** is the proportion of actual negatives that were correcty identified by the model.

$$
\text{Specificity} = \frac{TN}{TN + FP}
$$

High specificity means the model correctly rejects most negative cases.

It's complement is the **false positive rate (FPR)** which is the proportion of actual negatives that the model incorrectly identifies as positive.

$$
\text{FPR} = \frac{FP}{FP + TN} = 1 - \text{Specificity}
$$

### F1 Score

The F1 score is the harmonic mean of precision and recall.

$$
\text{F1} = 2 \cdot \frac{ab}{a+b} = \frac{2}{\frac{1}{b}+\frac{1}{a}}
$$

Since the smaller value dominates the contribution to the denominator, the harmonic mean is strongly affected by the smaller value.

Hence, a high F1 score requires both high precision and high recall. F1 is useful for imbalanced datasets when both false positives and false negatives matter, but it does not account for true negatives.

### Example

Suppose a disease-screening model produces the following results for 100 patients:

- $TP = 18$: sick patients correctly detected
- $TN = 72$: healthy patients correctly rejected
- $FP = 8$: healthy patients incorrectly flagged
- $FN = 2$: sick patients missed

The resulting metrics are:

$$
\begin{aligned}
\text{Accuracy} &= \frac{18 + 72}{100} = 0.90 \\
\text{Precision} &= \frac{18}{18 + 8} \approx 0.692 \\
\text{Recall} &= \frac{18}{18 + 2} = 0.90 \\
\text{Specificity} &= \frac{72}{72 + 8} = 0.90 \\
\text{F1} &= \frac{2(18)}{2(18) + 8 + 2} \approx 0.783
\end{aligned}
$$

Although the model has 90% accuracy and recall, its precision is only about 69.2% because several healthy patients are incorrectly flagged. This illustrates why accuracy alone does not fully describe classifier performance.

### Choosing a Metric

| Priority | Metric to emphasize |
| -------- | ------------------- |
| Overall correctness on balanced classes | Accuracy |
| Avoiding false positives | Precision or specificity |
| Avoiding false negatives | Recall / sensitivity |
| Balancing precision and recall | F1 score |

Precision and recall often trade off as the model's classification threshold changes. The appropriate metric therefore depends on the real-world costs of false positives and false negatives, not only on the metric's numerical value.
