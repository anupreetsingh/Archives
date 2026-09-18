## Types of Transformer Architectures

A **transformer** is a neural network that uses attention to build contextual representations of a sequence. For text, the sequence consists of **tokens**, which can represent words, parts of words, or punctuation. Attention lets each token draw information from other permitted positions in the sequence.

The three main architectural families are **encoder-only**, **encoder–decoder**, and **decoder-only**. They differ in which transformer blocks they use, which tokens can attend to one another, and how they produce an output.

All three use repeated blocks containing attention and feed-forward networks, together with residual connections and normalization. Positional information lets the model account for sequence order. Their typical training objectives are described below, but an architecture does not require one particular objective.

### Encoder-Only Architecture

An encoder-only transformer contains a stack of **encoder blocks** that turns an input sequence into contextual representations, usually one vector per input token.

Each block uses **bidirectional self-attention**: a token can attend to tokens both before and after it in the input. It is called self-attention because the interacting representations come from the same sequence. This lets the model interpret each token using the surrounding input as a whole.

These representations can feed a **task-specific output head** (a layer or small network that converts them into predictions).

For exampl:

- A classification head can use a sequence-level representation to predict sentiment.
- A token-classification head can assign a label to each token.

```text
Input: "The movie was excellent."
                    ↓
Encoder blocks: bidirectional self-attention + feed-forward networks
                    ↓
Contextual representations of the input tokens
                    ↓
            Classification head
                    ↓
            Sentiment: positive

Example pretraining task:
"The movie was [MASK]." → predict "excellent" at the masked position
```

A common pretraining objective is **masked language modeling**: hide selected input tokens and train the model to recover them using the visible context on both sides. The same encoder can then be adapted to downstream tasks using suitable training data and an output head.

**Typical uses:** Text classification, sentiment analysis, named-entity recognition, and extractive question answering, where the answer is selected from the input. Encoders trained for similarity or retrieval also produce useful representations for semantic search.

**Examples:** BERT and RoBERTa.

Encoder-only models are primarily used to represent and analyze an available input. Predicting missing tokens does not, by itself, give them the standard left-to-right generation process used by autoregressive decoders.

### Encoder–Decoder Architecture

An encoder–decoder transformer contains two stacks: an **encoder** that represents a source sequence and a **decoder** that generates a target sequence using those representations. This is the architecture of the original transformer and is commonly called a **sequence-to-sequence** model.

The encoder block works as described in the encoder section above. The decoder block her contains two attention operations before its feed-forward network:

- **Causal self-attention:** Each target position can attend to itself and earlier target positions, while a mask blocks later positions. The representation at that position is used to predict the next target token.
- **Cross-attention:** The decoder attends to the encoder's output representations. Its queries come from the decoder, while its keys and values come from the encoder. This lets each generation step retrieve relevant information from the source sequence.

The encoder therefore makes the source available as a sequence of contextual vectors; it does not need to compress the entire input into one vector. The target can have a different length from the source.

```text
Source: "The movie was excellent."
                    ↓
Encoder → contextual source representations
                    ↓ cross-attention at each generation step
Decoder
  <start>                         → "Le"
  <start> Le                      → "film"
  <start> Le film                 → "était"
  <start> Le film était           → "excellent"
  <start> Le film était excellent → "."
  <start> Le film était excellent. → <end>

Target: "Le film était excellent."
```

The example shows words for readability; actual generation proceeds in tokens.

During typical training, the decoder receives the correct target sequence shifted by one position and learns to predict the next target token. This is **teacher forcing**. The causal mask prevents access to the answer at future positions. During generation, the model instead feeds its own generated tokens back into the decoder, continuing until a stopping condition is reached.

**Typical uses:** Translation, summarization, and other tasks that transform a source sequence into a target sequence. Speech recognition can use this arrangement with an encoder designed to process audio features.

**Examples:** The original Transformer, T5, and BART.

### Decoder-Only Architecture

A decoder-only transformer contains a single stack of blocks using **causal self-attention**. In the standard text-only architecture, there is no separate encoder and no cross-attention to encoder outputs. The prompt and generated continuation form one sequence processed by the same stack.

The model performs **autoregressive generation**: it predicts a distribution over the next token from the available prefix, a token is selected, and that token is appended before the next prediction.

```text
Prompt: Translate to French: "The movie was excellent."
        Translation:

Prompt                              → predict "Le"
Prompt + Le                         → predict "film"
Prompt + Le film                    → predict "était"
Prompt + Le film était              → predict "excellent"
Prompt + Le film était excellent    → predict "."
Prompt + Le film était excellent.   → predict an end token
```

As in the previous example, words stand in for tokens. Generated tokens can attend to the preceding prompt and continuation through self-attention. There is no separate source representation supplied through cross-attention.

The usual pretraining objective is **next-token prediction**, also called causal language modeling. At each position, the model learns to predict the following token. Additional instruction tuning can train it to respond to requests expressed in the prompt.

Causal attention does not require training to process one token at a time. Because the training sequence is already known, predictions at many positions can be computed in parallel under the causal mask. Standard autoregressive generation is sequential because later output tokens depend on earlier generated tokens. This training-versus-generation distinction also applies to autoregressive encoder–decoder models.

**Typical uses:** Text completion, conversational assistants, code generation, and prompted tasks such as summarization and translation. The translation example shows that task capabilities overlap across architecture families.

**Examples:** GPT-style language models and Llama.
