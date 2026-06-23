# Chapter 5: The Prompt-as-Program View

## 5.1 Prompts as External Parameters (Program Inputs) to a Fixed Executor

The central thesis of this book is that a fixed-weight Transformer, when given a prompt, behaves as a **deterministic program interpreter**. The weights are the program; the prompt is the input. The output is the program's result.

This is a shift in perspective. The dominant view treats prompts as *queries* to a *black box*: you ask a question, the model generates an answer. The program view treats prompts as *code* for a *virtual machine*: you write instructions, the Transformer executes them.

**Analogy:**

| **Traditional Programming** | **Prompt Engineering** |
| :--- | :--- |
| CPU / VM | Fixed Transformer |
| Program (binary) | Model weights (fixed) |
| Program input (arguments) | Prompt (variable) |
| Executor (runtime) | Forward pass (attention + FFN + depth) |
| Output | Model output |

The program view enables us to think about prompts systematically:

- **Expressivity:** What functions can the fixed Transformer compute with different prompts? (Chapter 3)
- **Mechanism:** How are these functions computed? (Chapter 4)
- **Composition:** How can we combine prompts to perform complex tasks? (This chapter)
- **Verification:** How do we test that a prompt system works correctly? (Chapter 12)

---

## 5.2 A Formal Definition: Prompt Program as a 4-Tuple $(I, O, P, C)$

A prompt program is a formal object that specifies how a prompt is constructed and interpreted. We define it as a 4-tuple:

$$ \text{Prompt Program} = (I, O, P, C) $$

Where:

| **Component** | **Definition** | **Example** |
| :--- | :--- | :--- |
| **$I$** | Input space: the set of possible user queries | User question, document to summarize, code to debug |
| **$O$** | Output space: the set of possible responses | Answer, summary, fixed code, structured JSON |
| **$P$** | Prompt template: a function $P: I \to \text{Tokens}$ that maps inputs to prompts | "Let's think step by step about {query}" |
| **$C$** | Constraint set: a collection of constraints that $P$ and $O$ must satisfy | Format: JSON; Length: ≤ 100 words; Tone: neutral; Confidence: required |

The model $M$ with fixed weights $W$ computes:
$$ O = M( P(I) ) $$

The constraint set $C$ must be satisfied by $O$:
$$ \forall c \in C, \ c(O) = \text{true} $$

**Implications:**
- Prompt engineering is the process of designing $P$ and $C$ for a given $I$ and desired $O$.
- The model $M$ is fixed; the only variables are $P$ and $C$.
- The constraint set $C$ is often implicit in practice (e.g., "the answer should be correct"). The book makes it explicit.

---

## 5.3 Dependent Types and Probabilistic Refinements for Constraints

The constraints $C$ can be formalized using **dependent types**—types that depend on values. This allows us to express constraints like:

| **Constraint Type** | **Dependent Type Notation** | **Meaning** |
| :--- | :--- | :--- |
| **Output Schema** | $\text{JSON}(S)$ | Output must be valid JSON conforming to schema $S$ |
| **Length** | $\text{Length}_{\min, \max}(O)$ | Output length between min and max tokens |
| **Inclusion** | $\text{Contains}(k, O)$ | Output must contain keyword or pattern $k$ |
| **Confidence** | $\text{Confidence}(O) \in [0,1]$ | Output must include a confidence score |
| **Topic** | $\text{Topic}(O) \subseteq T$ | Output must be on topics within $T$ |
| **Exclusion** | $\text{NotContains}(k, O)$ | Output must not contain keyword or pattern $k$ |
| **Tone** | $\text{Tone}(O) \in \mathcal{T}$ | Output must have a specified tone (neutral, formal, etc.) |

**Probabilistic refinements** extend these with uncertainty:
- $\text{Length}_{\min,\max}(O)$ with probability $\ge 0.95$.
- $\text{Confidence}(O) \in [0.8, 1.0]$ with high confidence.

**Practical implication:** A well-designed prompt program is not just a string—it is a *typed specification* with explicit constraints. This is the foundation for type-driven programming (Chapter 10).

---

## 5.4 The Computational Model: Attention Routers + FFN Operators

The prompt program $(I, O, P, C)$ is interpreted by the Transformer. We can model this interpretation as a two-stage process:

### Stage 1: Attention as Routing

The attention mechanism routes information from the prompt $P(I)$ to the context. This is a **selection operator**:
$$ \text{Attn}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V $$

The prompt $P(I)$ provides the keys $K$ and values $V$. The context provides the queries $Q$. The attention output is a weighted sum of the prompt values—a *routed* representation.

### Stage 2: FFN as Operator

The routed representation is then passed through the FFN, which applies a non-linear transformation:
$$ \text{FFN}(x) = \text{ReLU}(xW_1 + b_1) W_2 + b_2 $$

The FFN acts as a **local operator** on each token independently. It is conditioned on the routed prompt information.

### Composition

With $L$ layers, the overall computation is:
$$ M(P(I)) = \text{FFN}_L \circ \text{Attn}_L \circ \dots \circ \text{FFN}_1 \circ \text{Attn}_1 (E(P(I))) $$

Where $E$ is the embedding function. This is a **composition of routers and operators**.

---

## 5.5 Semantic Compression via Naming and Abbreviation

**Formalization:** Naming and abbreviation are *compression operators* on the prompt space. They map an expanded instruction set to a smaller token sequence, while preserving semantic content through **semantic pointers**.

A semantic pointer is a symbol (word or abbreviation) that retrieves an entire compressed structure from the model's embedding space or from explicit definition in the prompt.

### The Compression-Decompression Cycle

1. **Decompression:** Define the abbreviation explicitly (e.g., "CoT" = "Let's think step by step").
2. **Compression:** Use the abbreviation in the prompt.
3. **Interpretation:** The model's attention retrieves the decompressed structure via the abbreviation's embedding.

### Hit Rate Model

The success of an abbreviation is determined by:
$$ \text{Hit Rate} = \text{Embedding Proximity} \times \text{Pretraining Frequency} $$

- **Embedding Proximity:** How close the abbreviation's embedding is to the concept it represents.
- **Pretraining Frequency:** How often the abbreviation appears in the model's training data.

### The Over-Compression Trap

Cryptic abbreviations (e.g., single letters) have low embedding proximity and low pretraining frequency. They fail to retrieve the correct structure.

**Rule:** Use abbreviations that appear frequently in the model's training corpus (e.g., "API", "JSON", "CoT") or define them explicitly before use.

### The Final Test

Replace the abbreviation with its expanded form. If the output quality drops, the abbreviation successfully transferred the pattern via compression. If the output improves, you over-compressed and lost signal.

**Engineering Rule:** Treat every name/abbrev as a cache key. Use them to compress context windows, never to obscure meaning.

---

## 5.6 Implications for Steer Sets: Pre-Compiling Frequently Used Components

A steer set (introduced in Chapter 8) is a collection of prompt components. Naming and abbreviation allow us to **pre-compile** frequently used components into compressed forms.

**Example:**
- Component E2 (Verification) can be abbreviated to "V&V".
- Component S3 (Aggressive Counter-Argumentation) can be abbreviated to "ACA".

These abbreviations act as cache keys in the prompt. They reduce token cost while preserving semantic fidelity.

**Implementation in STEER-CONTRACT:**
```yaml
components:
  - id: "E2"
    type: "Epistemic"
    description: "Verify own work, double-check all facts. Never hallucinate."
    compression:
      abbrev: "V&V"
      expansion: "Verification & Validation: fact-check, source-cite, confidence-score"
      hit_rate_estimate: 0.85
```

---

## 5.7 Chapter Summary

- Prompt-as-Program: A prompt is an external parameter to a fixed Transformer program.
- Formal Definition: A prompt program is a 4-tuple $(I, O, P, C)$.
- Constraints: Formalized via dependent types and probabilistic refinements.
- Computational Model: Attention as routing, FFN as operator, depth as composition.
- Semantic Compression: Abbreviations act as semantic pointers—cache keys to compressed structures.
- Hit Rate: Embedding proximity × pretraining frequency.
- Engineering Rule: Use abbreviations as cache keys; never obscure meaning.
- Steer Sets: Pre-compile frequently used components into abbreviations.

---

References Cited in This Chapter

· Kim et al. (2025). "Theoretical Foundations of Prompt Engineering: From Heuristics to Expressivity." arXiv:2512.12688.
· Nakada, R., et al. (2025). "A Theoretical Framework for Prompt Engineering: Approximating Smooth Functions with Transformer Prompts." arXiv:2503.20561.

```
