# AI Specific Technologies

AI applications are usually built as a stack. Lower layers handle data and math, middle layers train or run models, and higher layers connect models to tools, documents, databases, and user-facing applications.

## 1. Data and Scientific Python Foundation

These libraries form the base layer for most Python AI work.

### NumPy

**NumPy** provides fast numerical arrays and matrix operations.

It is the foundation of scientific Python because many ML libraries use NumPy arrays, or NumPy-like tensor concepts, internally.

Use NumPy when you need:

- Arrays and matrices
- Vectorized mathematical operations
- Linear algebra
- Numerical data preparation before ML

### Pandas

**Pandas** is used for tabular data analysis.

It is like a programmable spreadsheet for Python. It lets you load, clean, filter, group, join, and transform structured data before it is used by a model.

Use Pandas when you need:

- Loading and working with table data from sources like CSV, Excel, and SQL
- Data cleaning
- Feature preparation
- Exploratory Data Analysis(EDA)

### SciPy

**SciPy** builds on NumPy and provides scientific algorithms.

It is useful when the problem needs advanced mathematics beyond basic array operations.

Use SciPy when you need:

- Optimization
- Statistics
- Signal processing
- Scientific computing algorithms

## 2. Classical Machine Learning

Classical machine learning usually means algorithms that work well on structured data without needing huge neural networks.

### scikit-learn

**scikit-learn** provides ready-made classical ML algorithms.

It commonly sits after NumPy and Pandas in the workflow: Pandas prepares the dataset, NumPy represents numerical data, and scikit-learn trains or evaluates the model.

Common uses:

- Classification
- Regression
- Clustering
- Dimensionality reduction
- Train/test splitting
- Model evaluation

Example flow:

```text
CSV file -> Pandas DataFrame -> cleaned features -> scikit-learn model -> predictions
```

## 3. Visualization

Visualization helps understand data, debug model behavior, and communicate results.

### Matplotlib

**Matplotlib** is the basic graphing library for Python.

It is commonly used for static plots like line charts, scatter plots, histograms, and training curves.

### Plotly

**Plotly** is used for interactive visualizations.

It is common in dashboards and notebooks where the user needs to zoom, hover, filter, or inspect data interactively.

Relationship:

```text
Matplotlib -> simple static plots
Plotly -> interactive charts and dashboards
```

## 4. Deep Learning Libraries

Deep learning libraries are different from NumPy and classical machine learning libraries because they are built for training large neural networks.

NumPy gives you arrays for numerical computation, but it does not directly provide the full deep learning workflow. Classical ML libraries like scikit-learn give you ready-made algorithms, but they are not mainly designed for defining large neural network architectures layer by layer.

Deep learning frameworks provide:

- **Tensors:** Multi-dimensional arrays, similar to NumPy arrays, but designed for neural network computation.
- **GPU acceleration:** Ability to run tensor operations on GPUs, which are much faster for large matrix operations.
- **Automatic differentiation:** Ability to automatically calculate gradients during training.
- **Neural network layers:** Building blocks like linear layers, convolution layers, activation functions, and attention layers.
- **Training tools:** Optimizers, loss functions, model saving/loading, distributed training, and deployment tools.

### PyTorch

**PyTorch** is a deep learning framework widely used in research and production. It was originally developed by Meta, but now is handled by PyTorch Foundation, which is part of the Linux Foundation.

PyTorch gives you **tensors**, GPU support, automatic differentiation, neural network modules, and training utilities.

It is popular because it feels close to normal Python while still supporting advanced neural network training. This makes it common in research, experimentation, and custom model development.

Use PyTorch when building:

- Neural networks
- Computer vision models
- Natural language models
- Research experiments
- Custom training loops

### TensorFlow and Keras

**TensorFlow** is Google's deep learning framework. equivalent to PyTorch with the difference being mostly stylistic. Tensorflow does have more of a productions oriented ecosystem.

**Keras** is a higher-level API commonly used with TensorFlow to define and train neural networks with less boilerplate.

| Feature | PyTorch | TensorFlow/Keras |
| --- | --- | --- |
| Main style | Python-like and flexible | More framework-driven, especially with Keras |
| Common strength | Research, experimentation, custom training loops | Production deployment, mobile/web deployment, standardized model workflows |
| High-level API | `torch.nn` | `keras` |
| Core data structure | `torch.Tensor` | `tf.Tensor` |
| Typical feel | Write normal Python code and debug directly | Define models through TensorFlow/Keras abstractions |

## 5. Model Hubs and Transformer Libraries

Modern AI applications often use pretrained models instead of training everything from scratch.

### Hugging Face Transformers

**Hugging Face Transformers** provides access to pretrained transformer models.

It connects the deep learning layer to practical AI tasks by making models easier to download, fine-tune, and run.

Common uses:

- Text generation
- Text classification
- Summarization
- Translation
- Embeddings
- Tokenization

## 6. Chat and Embedding Model Providers

Instead of running models directly, many applications call model providers through APIs or local runtimes.

### OpenAI

**OpenAI** provides hosted chat, reasoning, embedding, image, and speech models through APIs.

In an AI application, OpenAI is usually the model provider. The application sends prompts, text, images, or documents to the API and receives model outputs.

### Claude

**Claude** is Anthropic's family of chat and reasoning models.

It plays a similar role to OpenAI in application architecture: the app calls Claude as an external model provider.

### Ollama

**Ollama** runs open model weights locally.

It is useful when you want a local development model, local inference, or more control over where model execution happens.

Relationship:

```text
OpenAI / Claude -> hosted model APIs
Ollama -> local model runtime
```

## 7. Embeddings and Vector Databases

Many AI systems need to search by meaning instead of exact keywords. This is where embeddings and vector databases fit.

### Embeddings

An **embedding** is a numerical representation of text, images, or other data.

Similar meanings produce vectors that are close to each other in vector space. This lets applications search for semantically related documents.

Example:

```text
"reset my password" -> embedding vector
"forgot login credentials" -> nearby embedding vector
```

### Qdrant

**Qdrant** is an open source vector database.

It stores embeddings and supports similarity search. It is commonly used in retrieval-augmented generation systems.

### Pinecone

**Pinecone** is a managed vector database service.

It serves the same general role as Qdrant, but is provided as a hosted service.

Relationship:

```text
Documents -> embeddings -> Qdrant/Pinecone -> similar documents -> LLM prompt
```

## 8. AI Application Frameworks

AI frameworks help connect models, prompts, tools, memory, retrieval, documents, and workflows.

### LangChain

**LangChain** is a framework for building LLM applications.

It helps connect model calls with prompts, tools, retrievers, vector databases, and multi-step workflows.

Common uses:

- Chatbots
- Retrieval-augmented generation
- Tool-using agents
- Chains of model calls
- Document question answering

### LlamaIndex

**LlamaIndex** is a framework for connecting LLMs to external data sources.

It is especially useful for retrieval-augmented generation because it helps load documents, split them into chunks, create indexes, retrieve relevant context, and send that context to an LLM.

Use LlamaIndex when building:

- Document question answering
- Knowledge-base chatbots
- Retrieval-augmented generation systems
- Search over private notes, PDFs, databases, or company data

Relationship:

```text
Documents/data -> LlamaIndex -> embeddings/index/retrieval -> LLM prompt
```

### CrewAI

**CrewAI** is a framework for building multi-agent workflows.

It focuses on organizing multiple agents with roles, tasks, and collaboration patterns.

Relationship:

```text
LangChain -> general LLM application orchestration
LlamaIndex -> data and document retrieval for LLMs
CrewAI -> role-based multi-agent orchestration
```

## 9. Tool and Function Calling Protocols

Models become more useful when they can call tools instead of only generating text.

### MCP

**MCP(Model Context Protocol)** is an open protocol for connecting AI applications to external tools and data sources.

It helps standardize how an AI system can access tools, files, databases, APIs, and other external context.

In the stack, MCP usually sits between the AI application framework and the outside systems the model needs to use.

Relationship:

```text
LLM app -> MCP client -> MCP server -> external tool or data source
```

## 10. Sequential Build Path

A typical learning and application-building sequence looks like this:

1. Learn **NumPy** for arrays and numerical operations.
2. Learn **Pandas** for loading, cleaning, and preparing datasets.
3. Use **Matplotlib** and **Plotly** to inspect and explain data.
4. Use **scikit-learn** for classical machine learning on structured data.
5. Learn **PyTorch** or **TensorFlow/Keras** for deep learning.
6. Use **Hugging Face Transformers** to work with pretrained transformer models.
7. Use **OpenAI**, **Claude**, or **Ollama** when the application needs chat or embedding models.
8. Store embeddings in **Qdrant** or **Pinecone** for semantic search.
9. Use **LlamaIndex** when the application needs to connect an LLM to documents, indexes, and retrieval.
10. Use **LangChain** or **CrewAI** to orchestrate prompts, tools, agents, retrieval, and workflows.
11. Use **MCP** when the AI application needs a standardized way to connect to external tools and context.

## 11. Common Application Patterns

### Classical ML Pipeline

```text
Pandas -> NumPy -> scikit-learn -> Matplotlib/Plotly
```

Use this for structured prediction problems like churn prediction, fraud detection, demand forecasting, and classification.

### Deep Learning Pipeline

```text
NumPy/Pandas -> PyTorch or TensorFlow/Keras -> trained neural network -> predictions
```

Use this when the model needs to learn complex patterns from images, text, audio, or large-scale data.

### LLM Application Pipeline

```text
User input -> LangChain/LlamaIndex/CrewAI -> OpenAI/Claude/Ollama -> response
```

Use this for chatbots, assistants, agents, summarizers, and reasoning workflows.

### Retrieval-Augmented Generation Pipeline

```text
Documents -> LlamaIndex -> embeddings -> Qdrant/Pinecone -> retrieved context -> OpenAI/Claude/Ollama -> answer
```

Use this when the model needs to answer using private documents, notes, company knowledge, or frequently changing information.

### Tool-Using Agent Pipeline

```text
User request -> LLM app -> MCP -> tool/API/database -> result -> LLM response
```

Use this when the model needs to perform actions or fetch live context from external systems.
