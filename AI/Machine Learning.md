# Machine Learning

## 1. Foundations

_Introduce how machines learn patterns from data and the core ideas needed to understand that process._

### Types of Machine Learning

_Compare supervised, unsupervised, self-supervised, and reinforcement learning._

### Essential Mathematics

_Review the linear algebra, probability, statistics, and calculus used by learning algorithms._

### Features, Labels, and Predictions

_Explain how inputs, expected outputs, and model predictions form a learning problem._

### Parameters, Hyperparameters, and Models

_Distinguish learned values, user-chosen settings, and the function produced by training._

## 2. The Machine-Learning Workflow

_Follow a typical ML project from defining the problem to testing a trained model._

### Data Collection and Preparation

_Cover data quality, missing values, outliers, labeling, encoding, scaling, and feature engineering._

### Train, Validation, and Test Sets

_Explain how separate datasets support learning, model selection, and unbiased evaluation._

### Training and Inference

_Distinguish learning model parameters from using the trained model to make predictions._

### Generalization and Overfitting

_Understand whether a model has learned useful patterns or merely memorized its training data._

## 3. Fundamental Supervised Algorithms

_Study algorithms that predict known categories or numerical targets from labeled examples._

### Linear and Logistic Regression

_Use weighted feature combinations for numerical prediction and probabilistic classification._

### k-Nearest Neighbors

_Make predictions using the labels or values of the most similar training examples._

### Naive Bayes

_Build probabilistic classifiers using Bayes' theorem and simplifying independence assumptions._

### Decision Trees

_Create interpretable prediction rules by repeatedly splitting data using useful features._

### Random Forests and Gradient Boosting

_Combine multiple trees through bagging or sequential error correction to improve predictions._

### Support Vector Machines

_Find maximum-margin decision boundaries and extend them to nonlinear problems with kernels._

## 4. Unsupervised Learning

_Discover groups, structure, and useful representations in data without target labels._

### Clustering

_Compare k-means, hierarchical clustering, and density-based methods for grouping similar observations._

### Dimensionality Reduction

_Use methods such as PCA to compress data while preserving important structure._

### Anomaly Detection

_Identify observations that differ significantly from patterns found in normal data._

## 5. Training and Evaluation

_Learn how model parameters are optimized and how predictive performance is measured._

### Loss Functions and Gradient Descent

_Measure prediction error and iteratively update parameters to reduce it._

### Regularization

_Control model complexity to reduce overfitting and improve performance on unseen data._

### Cross-Validation and Hyperparameter Tuning

_Compare model configurations reliably across repeated data splits._

### Evaluation Metrics

_Choose suitable regression or classification metrics based on the goal and costs of mistakes._

## 6. Neural-Network Fundamentals

_Understand how layers of artificial neurons learn nonlinear representations._

### Artificial Neurons and Layers

_Combine weighted inputs, biases, and activation functions into layered computations._

### Forward Propagation

_Trace an input through the network to produce a prediction and calculate its loss._

### Backpropagation

_Use the chain rule to calculate how each parameter contributed to prediction error._

### Optimizers

_Compare stochastic gradient descent, momentum, RMSProp, and Adam for updating parameters._

### Training Deep Networks

_Cover activations, initialization, normalization, dropout, learning rates, and unstable gradients._

## 7. Major Neural Architectures

_See how different network structures are adapted to particular kinds of data._

### Convolutional Neural Networks

_Use learned filters and spatial structure to process images and grid-like data._

### Recurrent Neural Networks, LSTMs, and GRUs

_Process sequences while preserving relevant information from earlier time steps._

### Autoencoders

_Learn compact representations by encoding inputs and then reconstructing them._

### Generative Adversarial Networks

_Train competing generator and discriminator networks to synthesize realistic samples._

## 8. Attention and Transformers

_Build from attention to the architecture behind modern language and multimodal models._

### Why Attention Emerged

_Explain how attention improves long-range information access and parallel sequence processing._

### Queries, Keys, and Values

_Interpret attention as matching queries with keys and combining the associated values._

### Self-Attention and Multi-Head Attention

_Build contextual token representations through several learned attention patterns._

### Positional Encoding

_Add ordering information that self-attention does not represent on its own._

### Transformer Blocks, Encoders, and Decoders

_Combine attention and feed-forward layers into architectures for understanding and generation._

## 9. Modern Models and Practical ML

_Connect transformer training to current foundation models and their real-world use._

### Tokenization and Embeddings

_Convert raw inputs into tokens and continuous vector representations used by models._

### BERT, GPT, and T5 Model Families

_Compare encoder-only, decoder-only, and encoder-decoder transformer designs._

### Pretraining, Fine-Tuning, and Alignment

_Learn general capabilities at scale and adapt them to tasks, instructions, and preferences._

### Retrieval and Multimodal Models

_Ground generation in external knowledge and extend models across text, images, audio, and video._

### Responsible Deployment

_Cover interpretation, bias, privacy, robustness, monitoring, drift, and retraining._
