# Foundations of Prompt Engineering

**From First Principles to Steerable Systems**

An open-source foundational theory textbook for prompt engineering.

---

## About This Book

This book establishes prompt engineering as a rigorous engineering discipline by:

1. **Proving and mechanistically explaining** the expressivity of fixed-weight Transformers under prompt variation (Prompt-UAT and structural decompositions).
2. **Introducing a formal 4-category taxonomy** of prompt components with algebraic operations (composition, override, conditional activation, priority resolution).
3. **Developing meta-level control theory** for steerable systems—modular, versioned, toggleable "steer sets" that enable reliable, auditable, and optimizable prompt programs.
4. **Formalizing semantic compression** via naming and abbreviation as a cache-key mechanism that enables 4x density gains while preserving semantic fidelity through careful anchor selection.

It is not a practical guide to "good prompts." It is a foundational theory text that explains *why* prompts work and *how* to systematically design, categorize, and control them.

---

## Target Audience

- Advanced undergraduate or graduate students in computer science, AI, or computational linguistics
- AI researchers and practitioners seeking a unified framework
- Prompt engineers who want to move beyond heuristics to principled design

**Prerequisites:** Basic machine learning concepts (neural networks, attention mechanisms). Working knowledge of Python is helpful for implementation chapters.

---
## Repository Structure

```

foundations-prompt-engineering/
├── README.md
├── LICENSE
├── manuscript/
│   ├── 00-preface.md
│   ├── 01-introduction.md
│   ├── 02-heuristics-gap.md
│   ├── 03-expressivity.md
│   ├── 04-mechanism-decomposition.md
│   ├── 05-prompt-as-program.md
│   ├── 06-taxonomy.md
│   ├── 07-constraint-catalog.md
│   ├── 08-steerable-systems.md
│   ├── 09-meta-prompting.md
│   ├── 10-type-driven-programming.md
│   ├── 11-building-systems.md
│   ├── 12-evaluation.md
│   ├── 13-deployment.md
│   ├── 14-reasoning-models.md
│   ├── 15-multi-agent.md
│   ├── 16-open-research.md
│   ├── appendices/
│   │   ├── A-formal-definitions.md
│   │   ├── B-steer-contract-spec.md
│   │   ├── C-exercises.md
│   │   └── D-glossary.md
│   └── references.md
└── steer-contract/
├── README.md
└── steer_contract_schema.yaml

```

---

## How to Contribute

This is a living document. Contributions are welcome:

- **Corrections**: Fix typos, factual errors, or broken citations.
- **Extensions**: Add new categories, constraints, or case studies.
- **Examples**: Provide concrete use cases for steer sets.
- **Implementations**: Build the STEER-CONTRACT parser, resolver, or evaluator.

Open an issue or submit a pull request.

---

## License

MIT License. See `LICENSE` file for details.

---

## Citation

If you use this book in your work, please cite:

```bibtex
@book{prompt-engineering-foundations,
  title={Foundations of Prompt Engineering: From First Principles to Steerable Systems},
  author={[Your Name]},
  year={2026},
  url={https://github.com/[your-username]/foundations-prompt-engineering}
}
```

---

Status

In progress. Chapters are being written and released incrementally. Current draft covers the theoretical foundations, taxonomy, and meta-level control theory. Practical chapters and the STEER-CONTRACT implementation are forthcoming.

---

Acknowledgments

Built on the formal foundations established by Kim et al. (2025), Petrov et al. (ICML 2024), Nakada et al. (2025), and the comprehensive taxonomy of Schulhoff et al. (2024).

```

