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

### ROC - AUC

Many classifiers produce a score or probability rather than a final class. A **classification threshold** converts that score into a label.

Lower threshold → more cases predicted positive → recall usually increases, precision usually decreases.

Higher threshold → fewer cases predicted positive → precision usually increases, recall usually decreases.

The **Receiver Operating Characteristic (ROC) curve** shows this tradeoff across every possible threshold. It plots:

- the **true positive rate (recall)** on the vertical axis;
- the **false positive rate ($1-\text{specificity}$)** on the horizontal axis.

Each point on the curve represents a different threshold. The ideal point is the top-left corner, where recall is $1$ and the false positive rate is $0$. A curve close to the diagonal line represents performance similar to random ranking.

![ROC curve](<Media/roc-threshold-tradeoff.svg>)

Moving from the bottom-left toward the top-right corresponds to lowering the threshold. Recall and the false positive rate increase because more examples are labeled positive; precision usually decreases because the newly included predictions tend to contain more false positives.

The **Area Under the ROC Curve (ROC-AUC)** summarizes the entire curve as a single threshold-independent value. It can be interpreted as the probability that the classifier assigns a higher score to a randomly selected positive example than to a randomly selected negative example.

- $\text{AUC}=1$: the model ranks every positive above every negative.
- $\text{AUC}=0.5$: its ranking is no better than random.
- $\text{AUC}<0.5$: its ranking is systematically reversed; swapping the ranking direction would improve it.

ROC-AUC is useful for comparing how well different models rank positive examples above negative examples across all possible thresholds. After choosing a model, its ROC curve can help select a classification threshold. The ideal operating point is the top-left corner, where the true positive rate is $1$ and the false positive rate is $0$. In practice, this point may not be achievable, so we choose a threshold corresponding to a point toward the top-left based on the acceptable trade-off between missed detections and false alarms.

However, ROC-AUC itself does not identify the best operating threshold, measure probability calibration, or account for the real-world costs of false positives and false negatives. It can also appear optimistic when the positive class is extremely rare because a large number of true negatives can keep the false positive rate small. In that situation, a **precision-recall curve and PR-AUC** often describe performance on the positive class more clearly.

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

## Regression Metrics

For regression scenarios, evaluation metrics measure how far the model's numerical predictions are from the true values. For an observation $i$, the **residual** or prediction error is:

$$
e_i = y_i - \hat{y}_i
$$

where $y_i$ is the actual value and $\hat{y}_i$ is the predicted value. A positive residual means the model underpredicted, while a negative residual means it overpredicted. Because positive and negative residuals could otherwise cancel out, MAE, MSE, and RMSE measure their magnitude instead.

### MAE

Mean Absolute Error (MAE) is the average absolute difference between the actual and predicted values.

$$
\text{MAE} = \frac{1}{n}\sum_{i=1}^{n}\left|y_i - \hat{y}_i\right|
$$

MAE is expressed in the same units as the target. For example, an MAE of \$3,000 on house-price predictions means the predictions are off by \$3,000 on average, without regard to direction.

> Every error contributes in direct proportion to its size

So MAE is less sensitive to outliers than MSE or RMSE. It is useful when each unit of error has roughly the same cost. However, the absolute value is not differentiable when the residual is zero, which can make it less convenient for some optimization methods.

### MSE

Mean Squared Error (MSE) is the average squared difference between the actual and predicted values.

$$
\text{MSE} = \frac{1}{n}\sum_{i=1}^{n}\left(y_i - \hat{y}_i\right)^2
$$

Squaring makes large errors contribute disproportionately and hence it is more sensitive to outliers. A $2$ contributes only $4$ whereas an error of $10$ contributes $100$.

MSE is therefore useful when large mistakes should be penalized heavily, and its smooth mathematical form makes it convenient as a training loss.

Its value is expressed in squared target units, such as dollars squared, so it is less directly interpretable than MAE or RMSE. It is also strongly affected by outliers.

### RMSE

Root Mean Squared Error (RMSE) is the square root of MSE.

$$
\text{RMSE} = \sqrt{\frac{1}{n}\sum_{i=1}^{n}\left(y_i - \hat{y}_i\right)^2}
$$

RMSE retains MSE's extra penalty for large errors but returns the result to the target's original units. It can therefore be interpreted as a typical error magnitude, with greater emphasis on large mistakes than MAE gives them.

For the same predictions, $\text{RMSE} \geq \text{MAE}$, with equality only when all absolute errors are equal. A large gap between RMSE and MAE often indicates that a few predictions have much larger errors than the rest. Because the square root is monotonic, MSE and RMSE always rank models in the same order when evaluated on the same observations.

### R^2

The coefficient of determination, $R^2$, measures a model's squared-error performance relative to a baseline that always predicts the mean of the actual values.

$$
R^2 = 1 - \frac{\sum_{i=1}^{n}\left(y_i - \hat{y}_i\right)^2}
{\sum_{i=1}^{n}\left(y_i - \bar{y}\right)^2}
$$

The numerator is the model's sum of squared errors, while the denominator is the mean-baseline's sum of squared errors.

- $R^2 = 1$: the predictions are perfect.
- $R^2 = 0$: the model is no better, by squared error, than always predicting the observed mean.
- $R^2 < 0$: the model is worse than that mean baseline.

$R^2$ is unitless and describes relative fit rather than the typical size of an error. For example, $R^2 = 0.80$ means the model accounts for 80% of the target's variation relative to the mean baseline; it does not mean that the model is 80% accurate. What counts as a good value depends on the problem, and a high $R^2$ does not establish that the model is unbiased, causal, or appropriate for unseen data. $R^2$ is undefined when all actual target values are identical because the denominator is zero.

### Example

Suppose a model produces four predictions:

| Actual value ($y_i$) | Predicted value ($\hat{y}_i$) | Residual ($y_i-\hat{y}_i$) | Absolute error | Squared error |
| -------------------- | ------------------------------ | --------------------------- | -------------- | ------------- |
| $3$ | $2.5$ | $0.5$ | $0.5$ | $0.25$ |
| $-0.5$ | $0$ | $-0.5$ | $0.5$ | $0.25$ |
| $2$ | $2$ | $0$ | $0$ | $0$ |
| $7$ | $8$ | $-1$ | $1$ | $1$ |

The absolute errors sum to $2$, the squared errors sum to $1.5$, and the actual-value mean is $\bar{y}=2.875$. The resulting metrics are:

$$
\begin{aligned}
\text{MAE} &= \frac{2}{4} = 0.5 \\
\text{MSE} &= \frac{1.5}{4} = 0.375 \\
\text{RMSE} &= \sqrt{0.375} \approx 0.612 \\
R^2 &= 1 - \frac{1.5}{29.1875} \approx 0.949
\end{aligned}
$$

MAE and RMSE describe the error in the target's units, while $R^2$ shows that the predictions reduce squared error by about 94.9% relative to predicting the mean for every observation.

### Choosing a Metric

| Priority | Metric to emphasize |
| -------- | ------------------- |
| An interpretable average error with limited outlier influence | MAE |
| Penalizing large errors heavily or using a smooth training loss | MSE |
| Penalizing large errors while retaining the target's units | RMSE |
| Measuring improvement over a mean-prediction baseline | $R^2$ |

MAE, MSE, and RMSE are **loss metrics**, so lower values are better and zero is perfect. $R^2$ is a **relative goodness-of-fit metric**, so higher values are better and one is perfect. In practice, reporting MAE or RMSE alongside $R^2$ gives both the real-world error magnitude and the improvement over a simple baseline.

Error magnitudes depend on the target's units and scale, so they should not be compared directly across unrelated datasets. Metrics should also be calculated on validation or test data rather than training data when the goal is to estimate performance on unseen examples.
