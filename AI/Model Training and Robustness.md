# Model Training and Robustness

This note develops the optimization and robustness topics that build on the basic training process in [Machine Learning](<Machine Learning.md#training>). It focuses on how batches affect gradient descent, why neural-network optimization can be difficult, how to stabilize training, and how to reason about models when production data differs from development data.

## Experimental Data Separation

The roles of the training, validation, and test sets are introduced in [Machine Learning](<Machine Learning.md#backpropagation-and-gradient>). Their separation must be preserved throughout development:

- The **training set** is used to learn model parameters.
- The **validation set** is used to choose hyperparameters, architectures, features, thresholds, and stopping points.
- The **test set** is used only after those choices have been made to estimate how the selected model performs on unseen data.

Repeatedly checking the test score and then changing the model based on it indirectly turns the test set into validation data. The model may not train on those test samples directly, but information from them has influenced development. The resulting test score is then an optimistic estimate rather than an independent evaluation.

Preprocessing transformations must follow the same separation. For example, normalization statistics, missing-value replacements, and feature-selection rules are fitted using the training set and then applied unchanged to the validation and test sets. This prevents [data leakage](<Machine Learning.md#preprocessing>).

## Gradient-Based Optimization

Gradient descent updates parameters in the direction that reduces the loss. For parameters \(\theta\), learning rate \(\eta\), and a set of samples \(B\), an update can be written as:

$$
\theta_{t+1} = \theta_t - \eta \frac{1}{|B|}\sum_{i \in B}\nabla_\theta L_i(\theta_t)
$$

The choice of \(B\) distinguishes the main gradient-descent variants.

### Full-Batch Gradient Descent

**Full-batch gradient descent** uses the entire training set to calculate one gradient and perform one parameter update.

- Each update follows the exact average training-set gradient.
- The gradient is stable and deterministic when the data and model state are unchanged.
- Each update can be slow and require too much memory for a large dataset.
- Only one parameter update occurs per epoch.

Full-batch training is practical for small datasets but is uncommon for large neural networks because the whole dataset may not fit in CPU or GPU memory.

### Stochastic Gradient Descent

**Stochastic gradient descent (SGD)** in its strict sense uses one randomly selected training sample for each parameter update.

- It requires little memory per update.
- It performs many inexpensive updates during an epoch.
- A single sample provides a noisy estimate of the full training-set gradient, so the loss can fluctuate instead of decreasing smoothly.
- The noise can help the optimizer move away from saddle points or narrow solutions, but excessive noise can make convergence unstable.

The name `SGD` is also commonly used for an optimizer that processes mini-batches. In that common usage, the more precise term is **mini-batch SGD**.

### Mini-Batch Gradient Descent

**Mini-batch gradient descent** divides the training set into batches containing more than one sample but fewer than the entire dataset. It is the standard approach for training neural networks.

- It fits large datasets into limited accelerator memory.
- It uses vectorized GPU operations more efficiently than processing one sample at a time.
- It updates parameters several times during each epoch.
- Its gradient is less noisy than a single-sample gradient but less expensive than a full-batch gradient.

The batch size controls a trade-off. Larger batches produce more stable gradient estimates but require more memory. Smaller batches introduce more gradient noise and more frequent updates. This noise can act as a form of **implicit regularization**, although it does not guarantee better generalization.

| Method | Samples per update | Updates per epoch | Gradient behavior | Typical use |
| --- | ---: | ---: | --- | --- |
| Full-batch gradient descent | Entire training set | 1 | Exact and stable | Small datasets |
| Stochastic gradient descent | 1 | Number of samples | Very noisy | Online learning or conceptual analysis |
| Mini-batch gradient descent | Chosen batch size | Number of samples divided by batch size | Moderately noisy | Most neural-network training |

### Data Order and Shuffling

Training data is normally shuffled before each epoch so that batches do not repeatedly preserve an accidental ordering in the dataset. Without shuffling, a dataset ordered by class, time, source, or difficulty could produce long sequences of biased gradient updates.

Shuffling does not change which samples the model sees. It changes the composition and order of mini-batches, helping each batch provide a more representative gradient estimate. Time-series and other sequence-dependent problems are exceptions when temporal order carries information and must be handled deliberately.

### Feature Scaling and Convergence

[Normalization and standardization](<Machine Learning.md#preprocessing>) prevent numerical features with large scales from dominating features with small scales. They also change the geometry of gradient-based optimization.

Suppose one feature usually ranges from \(0\) to \(1\), while another ranges from \(0\) to \(100{,}000\). The loss surface can become steep in one parameter direction and shallow in another. A single learning rate then causes the optimizer to oscillate across the steep direction while making slow progress along the shallow direction.

Scaling the features to comparable ranges makes the loss contours better conditioned, so gradient descent can take more direct and stable steps toward a minimum. Scaling is especially important for gradient-based models and distance-based algorithms. It is usually less important for decision trees because their splits depend on ordering rather than feature magnitude.

Scaling parameters such as the mean, standard deviation, minimum, and maximum must be calculated from the training set only.

## Non-Convex Optimization

A **convex** loss function has no suboptimal local minimum: every local minimum is also a global minimum. Many classical models have convex training objectives under suitable assumptions.

The loss functions of neural networks are generally **non-convex** because their layers and nonlinear activations compose many interacting parameters. Their optimization landscapes can contain:

- **Global minima:** parameter settings with the lowest possible loss.
- **Local minima:** parameter settings whose nearby alternatives have higher loss, even though a better solution may exist elsewhere.
- **Saddle points:** parameter settings where the loss curves upward in some directions and downward in others.
- **Flat regions and plateaus:** regions with gradients close to zero, causing slow progress.

Gradient descent is therefore not guaranteed to find a global minimum when training a neural network. In practice, finding the mathematical global minimum is often unnecessary: different parameter settings can achieve similar validation performance, and the solution that generalizes best does not necessarily have the lowest training loss.

### Parameter Initialization

The initial parameter values determine where optimization begins and can influence which region of a non-convex landscape the optimizer reaches. Initializing every weight to the same value is problematic because neurons then receive identical gradients and continue learning the same features.

Common schemes instead initialize weights randomly at a scale appropriate for the layer:

- **Xavier/Glorot initialization** is commonly paired with sigmoid or `tanh` activations.
- **He/Kaiming initialization** is commonly paired with ReLU-family activations.

These schemes try to keep activation and gradient magnitudes reasonably stable across layers. Training multiple runs with different random seeds can reveal whether performance is sensitive to initialization. Model selection must still use validation performance; the test set should not be used to choose the most favorable run.

## Exploding Gradients

Backpropagation applies the chain rule through successive layers. A simplified gradient through several transformations contains a product of derivatives:

$$
\frac{\partial L}{\partial h_1}
=
\frac{\partial L}{\partial h_n}
\prod_{k=2}^{n}
\frac{\partial h_k}{\partial h_{k-1}}
$$

If many of these factors have magnitudes greater than \(1\), their product can grow exponentially with network depth. This produces **exploding gradients**: extremely large gradients that cause oversized parameter updates, unstable or rapidly increasing loss, overflow, or `NaN` values.

Exploding gradients are especially associated with deep networks and recurrent networks, where the same transformation may participate repeatedly in backpropagation.

### Stabilization Techniques

**Gradient clipping** directly limits a gradient before the optimizer uses it. Global-norm clipping rescales the gradient vector when its norm exceeds a threshold \(c\):

$$
g_{\text{clipped}} =
\begin{cases}
g & \lVert g \rVert \le c \\
c\dfrac{g}{\lVert g \rVert} & \lVert g \rVert > c
\end{cases}
$$

Clipping preserves the gradient's direction while limiting the update magnitude. It controls the symptom but does not necessarily remove the underlying cause.

Other complementary techniques include:

- **Appropriate initialization:** Xavier or He initialization reduces the chance that signals grow immediately as they pass through layers.
- **Residual or skip connections:** A layer can learn a residual transformation \(F(x)\) while passing the input through a shorter path:

  $$
  y = F(x) + x
  $$

  The shorter path allows gradients to propagate without depending entirely on a long product of layer derivatives.
- **Normalization layers:** Batch normalization, layer normalization, and related methods can stabilize intermediate activation scales. They may reduce training instability but are not a guaranteed cure for exploding gradients.
- **A smaller learning rate:** This reduces the size of the parameter update produced by a large gradient, although an extremely large gradient may still require clipping or an architectural change.
- **A more suitable architecture or depth:** Gated recurrent units, LSTMs, residual networks, and shallower paths can improve gradient flow.

Gradient norms should be monitored during training so that clipping thresholds and other remedies respond to observed behavior rather than being chosen blindly.

## Problem Formulation

### Discretizing Regression as Classification

A problem with a continuous target is naturally expressed as regression, but the target can instead be divided into intervals and treated as a classification problem. This transformation is called **discretization** or **binning**.

For example, a continuous height target could be converted into categories:

| Height | Class |
| --- | --- |
| Less than 160 cm | Short |
| 160–180 cm | Medium |
| Greater than 180 cm | Tall |

Classification can be preferable when the downstream decision genuinely depends on categories, the measurements are too noisy to support useful exact predictions, or category-specific errors and thresholds matter more than numerical distance.

Binning also has costs:

- It discards information about differences within each interval.
- Values on opposite sides of a boundary receive different classes even when they are nearly identical.
- Values far apart inside the same interval are treated as equivalent.
- The number and placement of boundaries become additional design choices.

Therefore, binning should reflect the real decision being made rather than merely making the learning problem appear easier. When an exact quantity matters, regression usually retains more useful information. Ordinal classification methods can be appropriate when the categories have a meaningful order.

## Robustness Under Distribution Shift

The deployed-model monitoring section in [Machine Learning](<Machine Learning.md#monitoring>) defines data drift, concept drift, and performance degradation. These are production manifestations of a broader problem: development data and future inputs may come from different distributions.

### In-Distribution and Out-of-Distribution Data

**In-distribution (ID) data** resembles the distribution represented by the training and development data. **Out-of-distribution (OOD) data** differs in a meaningful way, such as inputs from a new location, time period, device, population, environment, or previously unseen category.

**OOD generalization** is a model's ability to maintain useful performance under such shifts. Good performance on a randomly sampled test set demonstrates generalization only to data drawn from approximately the same distribution. It does not by itself demonstrate robustness to future distribution changes.

### Decision Boundaries and Robustness

A classifier's **decision boundary** separates regions assigned to different classes. The **margin** of a sample is its distance from that boundary under a chosen representation and distance measure.

Small changes can flip predictions for samples very close to the boundary. A model that places representative samples farther from its boundary may be more robust to small local perturbations. However, a large margin on development data does not guarantee OOD robustness because a distribution shift can introduce inputs in entirely different regions or change the relationship between inputs and labels.

Decision-boundary robustness should therefore be treated as one property to evaluate, not as a substitute for testing realistic shifts.

### Evaluating OOD Generalization

OOD behavior can be evaluated using test sets that deliberately represent plausible production changes, such as:

- a later time period;
- a different geographic region or customer population;
- data from a different camera, sensor, or application version;
- rare conditions and categories that are important in production;
- controlled corruptions or perturbations relevant to the domain.

Performance should be reported separately for the original test distribution and each shifted condition. Prediction confidence and input-distribution monitoring can warn that something has changed when production labels are delayed, but a model can be confidently wrong. Ground-truth outcomes are ultimately needed to measure whether predictive performance has degraded and whether retraining or another intervention is necessary.
