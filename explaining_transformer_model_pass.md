## ✅ Pre-Forward Pass (Transformer Input Pipeline)

- Input text is entered into a client-facing application (e.g., ChatGPT) or sent via an API.

- The input text is **tokenized into tokens**  

- Each token is mapped to a **token ID**, which is used to **look up its embedding vector** from the model’s embedding matrix.

- The embedding vectors are **often scaled**, then combined with **positional encodings** so the model understands token order.

- The result is a **sequence of vectors (one per token)**, organized into batches.  
  This forms a tensor that is fed into the first transformer layer.

---

## 🧠 Key Insights

The transformer does **not** process:
- raw text  
- tokens  
- or token IDs  

It processes:

> **Dense vector representations of tokens (embeddings)**

**Tensor structure:**  
  The final input to the transformer has shape:
   *(batch_size, sequence_length, embedding_dim)*

## ✅ Forward Pass (Through a Transformer Layer)

- The input tensor is fed into the **first transformer layer**.

- Each layer takes the **output tensor from the previous layer** and applies a series of transformations.

---

### 🔹 Encoder vs Decoder Layers (Architecture Context)

- Transformer models are built using **encoder layers**, **decoder layers**, or both.

- **Encoder layers:**
  - Use **full self-attention**
  - Each token can attend to **all other tokens in the sequence**
  - Best for **understanding tasks** (classification, embeddings)

- **Decoder layers:**
  - Use **masked self-attention**
  - Each token can only attend to **previous tokens (not future ones)**
  - Enables **autoregressive generation (predicting next token)**

- Model examples:
  - GPT (Generative Pretrained Transformer) → **decoder-only**
  - BERT (Bidirectional Encoder Representations from Transformers) → **encoder-only**
  - Attention Is All You Need (original transformer) → **encoder + decoder**

---

### 🔹 Multi-Head Self-Attention

- The entire batch of token embeddings is processed in parallel.

- For each attention head:
  - The input embeddings are multiplied by three learned weight matrices to produce:
    - **Query (Q) vectors**
    - **Key (K) vectors**
    - **Value (V) vectors**

- Attention scores are computed:
  - A **dot product is taken between each token’s Query vector and all Key vectors** in the sequence.
  - This produces a **score for how much each token should attend to every other token**.

- The scores are:
  - **Scaled and passed through a softmax** to create attention weights.

- These weights are used to compute a weighted sum of the Value vectors:
  - This produces a **contextualized representation for each token**

> ⚠️ In decoder layers, this attention is **masked** so tokens cannot “see the future”

---

### 🔹 Combining Heads

- The outputs from all attention heads are:
  - **Concatenated together**
  - Then passed through a **final linear transformation**

---

### 🔹 Feed-Forward Network (FFN)

- Each token’s vector is passed independently through a small neural network:
  - Typically: **Linear → Activation → Linear**

---

### 🔹 Residual Connections + Normalization

- After both attention and FFN steps:
  - A **residual connection** (skip connection) is added
  - Followed by **layer normalization**

---

### 🔁 Layer Stacking

- This entire process repeats across multiple layers.
- Each layer produces a **more context-aware embedding for every token**

---

## 🔧 Key Clarifications from Your Original Notes

- **Attention is not just Q·K → V directly:**
  - Q and K produce **attention scores**
  - These scores determine how V vectors are combined

- **Each head sees the full sequence (encoder) or past sequence (decoder):**
  - Encoder → full visibility  
  - Decoder → masked (left-to-right)

- **Output is still one vector per token:**
  - But now enriched with context from other tokens in the sequence

---

## 🧠 Key Insight

Each layer doesn’t produce tokens — it produces:

> **A refined set of embedding vectors, where each token now “understands” its context relative to all other tokens.**

