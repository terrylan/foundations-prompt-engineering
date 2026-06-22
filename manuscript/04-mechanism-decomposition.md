# Chapter 4: Mechanism-Level Decomposition

## 4.1 The Transformer Architecture: A Brief Refresher

Before we can decompose how prompts interact with Transformers, we must establish a common vocabulary. This section provides a high-level refresher of the Transformer architecture (Vaswani et al. 2017). Readers already familiar with Transformers may skip to Section 4.2.

A Transformer consists of:

- **Input Embedding:** Tokens are mapped to continuous vectors in \( \mathbb{R}^d \).
- **Positional Encoding:** Sinusoidal or learned embeddings that encode token position.
- **L Layers:** Each layer has:
  - **Multi-Head Self-Attention:** Each head computes attention scores between tokens, producing a weighted sum of values.
  - **Feed-Forward Network (FFN):** A two-layer MLP applied to each token independently.
  - **Residual Connections:** Add the input to the output of each sub-layer.
  - **Layer Normalization:** Normalizes activations.
- **Output Projection:** The final layer's output is projected to token logits.

The key mechanisms for prompt engineering are:

1. **Attention:** Selectively routes information from prompt tokens to context tokens.
2. **Feed-Forward Networks:** Perform local arithmetic on the routed information.
3. **Depth:** Composes these operations across layers.

We examine each mechanism in detail.

---

## 4.2 Attention as Selective Routing from Prompt Memory

Attention is the primary mechanism by which prompts influence model behavior. It is a **routing mechanism** that allows the model to selectively retrieve information from the prompt and incorporate it into the context.

### The Attention Formula

For a single attention head, the computation is:
\[
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V
\]
where \( Q \) (queries) come from the context tokens, and \( K \) (keys) and \( V \) (values) come from the prompt tokens.

**Interpretation:**
- Each context token \( c \) produces a query vector \( q_c \).
- Each prompt token \( p \) produces a key vector \( k_p \) and value vector \( v_p \).
- The attention score \( a_{c,p} = \text{softmax}(q_c \cdot k_p / \sqrt{d_k}) \) measures the relevance of prompt token \( p \) to context token \( c \).
- The output is a weighted sum of the value vectors: \( \sum_p a_{c,p} v_p \).

### How Prompts Control Routing

By designing the prompt, we control the key and value vectors that the model attends to. This is achieved through:

1. **Token Choice:** Different tokens have different embeddings, producing different key/value vectors.
2. **Position:** Token position affects positional encoding, which influences attention patterns.
3. **Semantic Content:** The meaning of a token determines its embedding, which affects its key and value projections.

**Example:** A prompt that says "Verify each step" produces keys that are semantically associated with verification. The model attends to these tokens when processing a reasoning task, retrieving the "verification" behavior.

**Formal Insight (Kim et al. 2025):** The attention mechanism performs a **selective routing** from prompt memory. The prompt is a content-addressable memory that the model accesses to retrieve relevant information and procedures.

---

## 4.3 Feed-Forward Networks as Local Arithmetic Conditioned on Retrieved Fragments

After attention retrieves information from the prompt, the **Feed-Forward Network (FFN)** performs local arithmetic on the retrieved fragments.

### The FFN Formula

The FFN is a two-layer MLP:
\[
\text{FFN}(x) = \text{ReLU}(x W_1 + b_1) W_2 + b_2
\]
where \( x \) is the output of the attention mechanism for a single token.

### How Prompts Control Arithmetic

The FFN operates on each token independently. It cannot directly reference other tokens—it only sees its own input. However, that input is a *conditioned* representation: it includes information retrieved from the prompt via attention.

Thus, the FFN performs **conditional computation**:
- **Condition:** The attention output is a weighted sum of prompt value vectors.
- **Arithmetic:** The FFN applies a non-linear transformation to this conditioned input.

**Example:** For a math problem, the prompt "Let's think step by step" retrieves a set of value vectors that correspond to reasoning patterns. The FFN then transforms these into step-by-step arithmetic operations.

**Formal Insight (Kim et al. 2025):** The FFN performs **local arithmetic** on the retrieved fragments. The prompt's role is to *select which arithmetic operations are performed*.

---

## 4.4 Depth-Wise Stacking as Composition of Multi-Step Computations

A single attention + FFN layer performs one step of computation. By stacking \( L \) layers, the Transformer performs a **composition** of these steps.

### The Compositional View

Each layer \( l \) takes as input the output of the previous layer and produces a refined representation. The overall computation is:
\[
f(x) = f_L \circ f_{L-1} \circ \dots \circ f_1(x)
\]
where each \( f_l \) is a layer (attention + FFN + residual + layer norm).

### How Prompts Control Composition

The prompt affects the composition in two ways:

1. **Layer-wise Conditioning:** At each layer, attention retrieves relevant prompt information. This allows the prompt to influence computation at every stage, not just the first layer.

2. **Depth-Selective Routing:** Some prompt tokens may be attended to at early layers, others at later layers. This enables **hierarchical control**—the prompt can specify both low-level operations (early layers) and high-level reasoning (late layers).

**Example:** A prompt that says "First retrieve facts, then synthesize, then verify" can induce different attention patterns at different layers. Early layers attend to retrieval-related tokens; middle layers attend to synthesis tokens; late layers attend to verification tokens.

**Formal Insight (Kim et al. 2025):** Depth-wise stacking **composes** the local updates into a multi-step computation. The prompt is a program that specifies what each step should do.

---

## 4.5 How Prompts Interface with Each Mechanism

We can now summarize how prompts interface with each mechanism:

| **Mechanism** | **Role in Prompt Control** | **Prompt Design Implication** |
| :--- | :--- | :--- |
| **Embedding** | Maps tokens to vectors; determines initial representation. | Token choice matters. Synonyms may produce different embeddings. |
| **Attention** | Routes information from prompt to context. | Use key phrases that produce distinct key vectors. |
| **FFN** | Performs conditional arithmetic on retrieved fragments. | Provide information-rich prompts that give the FFN something to compute. |
| **Depth** | Composes multiple steps. | Structure prompts to specify a sequence of operations. |

---

## 4.6 Implications for Prompt Design

The mechanism-level decomposition provides practical guidance for prompt engineering:

1. **Use Distinct Key Phrases:** Tokens with similar embeddings produce similar keys. To route attention effectively, use semantically distinct terms.

2. **Provide Information-Rich Content:** The FFN can only compute on what attention retrieves. Vague prompts produce vague attention outputs.

3. **Structure Prompts Hierarchically:** Depth-wise composition means that early layers handle low-level detail, and late layers handle high-level reasoning. Split your prompt accordingly.

4. **Leverage Position:** Positional encoding affects attention patterns. Place key instructions at the beginning or end, depending on the model's positional bias.

5. **Test for Depth-Sensitivity:** Some tasks require more layers than others. If a prompt fails, try adding more structural detail to engage deeper layers.

---

## 4.7 Chapter Summary

- A Transformer consists of attention, FFN, and depth-wise composition.
- **Attention** is a routing mechanism: it selectively retrieves prompt information into the context.
- **FFN** is a conditional arithmetic unit: it transforms the retrieved information.
- **Depth** composes these operations: each layer refines the representation.
- Prompts interface with each mechanism: they control what information is retrieved, what arithmetic is performed, and how many steps are composed.
- Design implications: use distinct terms, provide rich content, structure hierarchically, consider position, and test for depth-sensitivity.

---

## References Cited in This Chapter

- Vaswani, A., et al. (2017). "Attention Is All You Need." *NeurIPS*.
- Kim et al. (2025). "Theoretical Foundations of Prompt Engineering: From Heuristics to Expressivity." *arXiv:2512.12688*.