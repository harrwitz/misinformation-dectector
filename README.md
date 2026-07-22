```markdown
<div align="center">
  
# 🔍 Non-LLM Misinformation Verifier
  
*An offline, deterministic NLP pipeline that verifies real-time news claims using classical NLP, sentence cross-encoders, and Natural Language Inference (NLI)—**completely free of Generative LLMs**.*

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Maintained](https://img.shields.io/badge/Maintained%3F-yes-brightgreen.svg)]()

</div>

<br />

## 📖 Table of Contents
- [Why Build This Without an LLM?](#-why-build-this-without-an-llm)
- [Architecture & Pipeline](#️-architecture--pipeline)
- [Key Technical Features](#-key-technical-features)
- [Quickstart](#-quickstart)
- [Example Outputs](#-example-outputs)
- [Tech Stack](#-tech-stack)
- [Contributing](#-contributing)
- [License](#-license)

---

## 💡 Why Build This Without an LLM?

Large Language Models (LLMs) are great for creative writing, but they come with real drawbacks for automated fact-checking:
* **Hallucinations:** Generative models can confidently invent facts or fabricate sources.
* **Latency & Cost:** Calling commercial APIs for simple stance detection is slow and expensive.
* **Non-Deterministic:** The same prompt can yield different verdicts on different runs.

This project proves you can build an accurate, real-time misinformation verifier using lightweight classification models, dependency parsing, and semantic search. **If there's no evidence in the news, it simply tells you—it will never invent a story.**

---

## 🛠️ Architecture & Pipeline

```text
  [ User Claim ]
        │
        ├──► 1. spaCy Dependency Parsing (Isolate core grammatical subject)
        ├──► 2. Query Generation (NLTK WordNet synonyms + noun/verb combinations)
        │
        ▼
  [ Google News Search ] ──► Gathers 15-20 candidate articles across multi-index passes
        │
        ▼
  [ Subject Guardrail ] ──► Discards sentences that don't explicitly mention the claim's subject
        │
        ▼
  [ Cross-Encoder Ranking ] ──► `ms-marco-MiniLM` ranks sentences by factual relevance
        │
        ▼
  [ Speculation Intercept ] ──► Detects journalistic hedging ("wants", "reportedly", "rumor")
        │                        └──► Flags premature claims as MISINFORMATION
        ▼
  [ RoBERTa NLI Stance ] ──► `roberta-large-mnli` classifies entailment vs. contradiction
        │
        ▼
  [ Final Verdict ]

```

---

## ✨ Key Technical Features

* **Grammatical Subject Guardrail:** Uses `spaCy` dependency parsing (`nsubj`) to ensure candidate sentences strictly contain the claim's subject. This prevents semantic search models from making "desperate matches" (e.g., substituting "vaping" for "smoking").
* **Cross-Encoder Fact Retrieval:** Replaces traditional vector-similarity search (SBERT) with an MS-MARCO Cross-Encoder. This guarantees candidate sentences actually *answer* or *resolve* the claim rather than just sharing similar words.
* **Journalistic Speculation Filter:** Intercepts claims before stance detection. If a user states something as a completed fact (e.g., *"Player X transferred to Club Y"*), but the news only reports on rumors or intent (*"Player X wants to move"*), the pipeline flags it as **Premature Misinformation**.
* **Synonym-Aware Query Expansion:** Combines proper nouns and verbs with WordNet synonyms to run multi-pass news searches, overcoming vocabulary mismatches across news outlets.

---

## 🚀 Quickstart

### 1. Prerequisites & Installation

Clone the repository and install the required NLP libraries:

```bash
git clone [https://github.com/your-username/misinformation-verifier.git](https://github.com/your-username/misinformation-verifier.git)
cd misinformation-verifier

pip install spacy nltk gnews sentence-transformers transformers torch
python -m spacy download en_core_web_sm

```

### 2. Run the Verifier

Launch the interactive CLI:

```bash
python verifier.py

```

---

## 🧪 Example Outputs

### Case 1: Caught Speculation / Unconfirmed Transfer

```text
CLAIM TO VERIFY: "olise moves to real madrid"
======================================================================
Executing search across 8 query variations...
Retrieved 20 distinct articles for analysis.
Analyzing sentences for exact factual resolution...

======================================================================
VERIFICATION SUMMARY
======================================================================
VERDICT:           ❌ MISINFORMATION (Premature Fact / Speculation Found)
CONFIDENCE:        0.0%
ARTICLES COMPARED: 20
BEST EVIDENCE:     "Michael Olise wants to move to Real Madrid - Yahoo Sports"
======================================================================

```

### Case 2: Subject Guardrail in Action

```text
CLAIM TO VERIFY: "smoking causes cancer"
======================================================================
Executing search across 8 query variations...
Retrieved 20 distinct articles for analysis.
Filtering out news that doesn't mention: {'smoking', 'smoke'}
Analyzing sentences for exact factual resolution...

======================================================================
VERIFICATION SUMMARY
======================================================================
VERDICT:           ✅ VERIFIED (Supported by News)
CONFIDENCE:        88.42%
ARTICLES COMPARED: 20
BEST EVIDENCE:     "Cigarette smoking causes roughly 85% of all lung cancer cases."
======================================================================

```

---

## 📦 Tech Stack

| Component | Library / Model | Purpose |
| --- | --- | --- |
| **Parsing & Syntactic Rules** | `spaCy` (`en_core_web_sm`) | POS tagging, dependency parsing (`nsubj`), entity isolation |
| **Synonym Expansion** | `NLTK` (`wordnet`) | Generating query variations for news search |
| **News Retrieval** | `gnews` | Real-time article headline and snippet aggregation |
| **Fact Ranking** | `sentence-transformers` (`ms-marco-MiniLM-L-6-v2`) | Cross-Encoder for sentence relevance scoring |
| **Stance Classification** | `transformers` (`FacebookAI/roberta-large-mnli`) | NLI logical entailment vs. contradiction evaluation |

---

## 🤝 Contributing

Pull requests are warmly welcome. If you'd like to improve news scraping capabilities (e.g., full-text scraping via BeautifulSoup) or extend the hedge word dictionary for different domains (politics, finance), please fork the repository and open a pull request.

---

## 📜 License

Distributed under the MIT License. See `LICENSE` for details.

```

```
