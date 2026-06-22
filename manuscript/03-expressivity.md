# Chapter 3: Expressivity of Fixed-Weight Transformers Under Prompt Variation

## 3.1 Universal Approximation in Neural Networks: Historical Context

The question "what can a neural network approximate?" has a long history. The classical Universal Approximation Theorem (Cybenko 1989; Hornik 1991) states that a feedforward neural network with a single hidden layer and a non-linear activation function can approximate any continuous function on a compact domain, given sufficient width.

This result is *existence-based*—it tells us that *some* network can approximate *some* function, but it does not tell us how to find the weights, nor does it provide practical bounds on width or depth.

The theorem was extended to deep networks, recurrent networks, and attention-based architectures. But the key insight for prompt engineering is different:

> *We are not asking what a Transformer can approximate with arbitrary weights. We are asking what a Transformer can approximate when the weights are fixed, and the only variable is the prompt.*

This is a different problem. It is the **Prompt Universal Approximation Theorem (Prompt-UAT)**.

---

## 3.2 Prompt-UAT: Formal Statement and Proof Sketch

The Prompt-UAT, formalized by Kim et al. (2025) and Petrov et al. (2024), states:

> **Informal Statement:** For any continuous function \( f: \mathcal{X} \to \mathcal{Y} \) on a compact domain, and for any fixed Transformer architecture with sufficient width and depth (but fixed weights), there exists a prompt \( p \) such that the model's output approximates \( f(x) \) within \( \epsilon \) for all \( x \in \mathcal{X} \).

**Formal Statement (Kim et al. 2025):**
Let \( T \) be a fixed Transformer with \( L \) layers, \( H \) attention heads, and hidden dimension \( d \). For any \( f \in \mathcal{C}^\beta(\mathcal{X}, \mathcal{Y}) \) (the space of \( \beta \)-times differentiable functions) and any \( \epsilon > 0 \), there exists a prompt \( p \in \mathcal{P} \) (the space of finite-length token sequences) such that:
\[
\| T(x \oplus p) - f(x) \| \le \epsilon \quad \forall x \in \mathcal{X}
\]
where \( \oplus \) denotes concatenation.

**Proof Sketch:**

The proof proceeds in three steps:

1. **Embedding:** The prompt \( p \) is tokenized and embedded into the model's hidden space. This creates a set of *query vectors* that can be accessed by attention heads.

2. **Routing:** The attention mechanism routes information from the prompt tokens to the context tokens. By designing the prompt to contain specific key–value pairs, we can control which information is attended to at each layer.

3. **Arithmetic:** The feed-forward networks perform local arithmetic on the routed information. By composing these operations across layers, we can approximate any continuous function.

The constructive proof shows that for any \( f \), we can design a prompt that acts as a *program*—a sequence of instructions that the Transformer executes layer by layer.

**Key Insight:** The prompt is not a query. It is a *program* that is *interpreted* by the fixed Transformer.

---

## 3.3 Prompt Length as a Complexity Parameter

The Prompt-UAT is an existence proof. It does not tell us how long the prompt must be.

Nakada et al. (2025) provide quantitative bounds. They show that for a Transformer with \( L \) layers and \( d \) hidden dimensions, the prompt length required to approximate \( f \in \mathcal{C}^\beta \) with error \( \epsilon \) is bounded by:
\[
\text{length}(p) \le \mathcal{O}\left(\epsilon^{-\frac{d}{\beta}}\right)
\]
where \( d \) is the input dimension and \( \beta \) is the smoothness of \( f \).

**Implications:**

| **Function Class** | **Smoothness \( \beta \)** | **Prompt Length Scaling** |
| :--- | :--- | :--- |
| Smooth (\( \beta \) large) | High | Sublinear in \( \epsilon^{-1} \) |
| Rough (\( \beta \) small) | Low | Exponential in \( \epsilon^{-1} \) |
| Extremely rough (\( \beta \to 0 \)) | Very low | Not approximable with finite prompts |

**Practical Interpretation:**

- For smooth functions (e.g., arithmetic, reasoning), moderate-length prompts (e.g., 100–1000 tokens) suffice.
- For highly non-smooth functions (e.g., arbitrary classification boundaries in high-dimensional spaces), prompt length may be impractical—prompt engineering cannot solve all problems.
- The bound is *constructive* but not *optimal*. Real-world prompts are often shorter than the bound suggests.

---

## 3.4 Limitations: Inductive Biases, Pretraining Distributions, and Practical Non-Universality

The Prompt-UAT is a mathematical result. It assumes:

- **Infinite model width/depth:** Real Transformers have finite \( L \), \( H \), \( d \).
- **Arbitrary prompt tokens:** Real prompts are constrained to natural language tokens.
- **No inductive biases:** Real Transformers are pretrained on text; they have strong preferences for certain patterns over others.
- **Compact domain:** Real inputs are unbounded and may not fit into a compact set.

These assumptions are violated in practice. The result is *existential*—it tells us that a prompt *exists* in principle, but it does not tell us if that prompt is:

- **Discoverable:** Can we find it via search or optimization?
- **Interpretable:** Is it a natural language string that humans can understand?
- **Stable:** Is it robust to input variations or model updates?

Moreover, pretrained Transformers have **inductive biases**—they are trained on text from the web, which biases them towards linguistic patterns, common sense, and specific cultural norms. These biases limit universality: a prompt that approximates a function in one domain (e.g., coding) may fail in another (e.g., medicine).

**Practical takeaway:** Prompt-UAT is a *ceiling*—it tells us what is theoretically possible. But real-world prompt engineering operates *below* that ceiling. The gap is what we must bridge with taxonomy, meta-control, and systematic practice.

---

## 3.5 Why This Matters for Prompt Engineering

The Prompt-UAT is not merely an academic curiosity. It has direct implications for practice:

1. **Justification:** It proves that prompt engineering is not a hopeless endeavor. There *exists* a prompt for almost any task—we just need to find it.

2. **Guidance:** It suggests that prompt length and complexity should scale with task difficulty. Harder tasks require more detailed prompts.

3. **Limitation:** It sets a ceiling. For tasks that are too complex or too non-smooth, prompt engineering will fail—we need fine-tuning or architecture changes.

4. **Framework:** It provides the theoretical foundation for the rest of this book. If prompts are programs, then we can design them systematically.

---

## 3.6 Chapter Summary

- Universal Approximation Theorems historically proved that neural networks can approximate any continuous function.
- Prompt-UAT extends this to fixed-weight Transformers: there exists a prompt for almost any function.
- The constructive proof decomposes the Transformer into attention (routing) and feed-forward (arithmetic) operations.
- Prompt length scales with task complexity: smooth functions require shorter prompts; rough functions require longer prompts.
- Limitations: finite model capacity, inductive biases, and practical constraints mean Prompt-UAT is a *ceiling*, not a *floor*.
- For prompt engineering, this result provides justification, guidance, limitation, and a theoretical foundation.

---

## References Cited in This Chapter

- Cybenko, G. (1989). "Approximation by superpositions of a sigmoidal function." *Mathematics of Control, Signals, and Systems*.
- Hornik, K. (1991). "Approximation capabilities of multilayer feedforward networks." *Neural Networks*.
- Kim et al. (2025). "Theoretical Foundations of Prompt Engineering: From Heuristics to Expressivity." *arXiv:2512.12688*.
- Petrov et al. (2024). "Universal in-context approximation by prompting fully recurrent models." *ICML 2024*.
- Nakada, R., et al. (2025). "A Theoretical Framework for Prompt Engineering: Approximating Smooth Functions with Transformer Prompts." *arXiv:2503.20561*.