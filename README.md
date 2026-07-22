```markdown
Non-LLM Misinformation Verifier

An offline, deterministic NLP pipeline that verifies real-time news claims using classical NLP, sentence cross-encoders, and Natural Language Inference (NLI)—completely free of Generative LLMs.


Table of Contents
- Why Build This Without an LLM?
- Architecture & Pipeline
- Key Technical Features
- How It Works
- Example Outputs
- Tech Stack
- Contributing
- License


Why Build This Without an LLM?

Large Language Models (LLMs) are great for creative writing, but they come with real drawbacks for automated fact-checking:
* Hallucinations: Generative models can confidently invent facts or fabricate sources.
* Latency & Cost: Calling commercial APIs for simple stance detection is slow and expensive.
* Non-Deterministic: The same prompt can yield different verdicts on different runs.

This project proves you can build an accurate, real-time misinformation verifier using lightweight classification models, dependency parsing, and semantic search. If there's no evidence in the news, it simply tells you—it will never invent a story.


Architecture & Pipeline

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
  [ Cross-Encoder Ranking ] ──► ms-marco-MiniLM ranks sentences by factual relevance
        │
        ▼
  [ Speculation Intercept ] ──► Detects journalistic hedging ("wants", "reportedly", "rumor")
        │                        └──► Flags premature claims as MISINFORMATION
        ▼
  [ RoBERTa NLI Stance ] ──► roberta-large-mnli classifies entailment vs. contradiction
        │
        ▼
  [ Final Verdict ]


Key Technical Features

* Grammatical Subject Guardrail: Uses spaCy dependency parsing (nsubj) to ensure candidate sentences strictly contain the claim's subject. This prevents semantic search models from making "desperate matches" (e.g., substituting "vaping" for "smoking").
* Cross-Encoder Fact Retrieval: Replaces traditional vector-similarity search (SBERT) with an MS-MARCO Cross-Encoder. This guarantees candidate sentences actually answer or resolve the claim rather than just sharing similar words.
* Journalistic Speculation Filter: Intercepts claims before stance detection. If a user states something as a completed fact (e.g., "Player X transferred to Club Y"), but the news only reports on rumors or intent ("Player X wants to move"), the pipeline flags it as Premature Misinformation.
* Synonym-Aware Query Expansion: Combines proper nouns and verbs with WordNet synonyms to run multi-pass news searches, overcoming vocabulary mismatches across news outlets.


How It Works

The system converts input claims into verified outcomes through a four-stage process:

1. Syntactic Parsing & Query Expansion
   The user inputs a claim. spaCy parses the sentence structure to identify the root verb and core grammatical subject (nsubj). NLTK WordNet extracts direct synonyms for these key terms to construct multiple distinct search queries.

2. Real-Time Retrieval & Filtering
   The generated queries are dispatched to Google News to pull candidate news headlines and descriptions. The Subject Guardrail immediately evaluates all gathered text, discarding any sentence that does not contain the target subject or its synonyms.

3. Cross-Encoder Fact Ranking
   Remaining sentences are paired directly with the claim and scored using a Cross-Encoder model (ms-marco-MiniLM). The Cross-Encoder computes full cross-attention between the claim and candidate sentences to surface the exact sentence containing the factual resolution.

4. Speculation Check & NLI Classification
   The selected best-matching sentence passes through a Speculation Filter to catch hedge words (such as "wants", "reportedly", or "rumor"). If speculation is detected on a factual assertion, the system flags it as premature misinformation. Otherwise, the pair is evaluated by a RoBERTa Natural Language Inference model (roberta-large-mnli) to output a final verdict of Verified, Misinformation, or Unverified based on strict logical entailment.


Example Outputs

Case 1: Caught Speculation / Unconfirmed Transfer

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


Case 2: Subject Guardrail in Action

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


Tech Stack

Component                      Library / Model                  Purpose
Parsing & Syntactic Rules      spaCy (en_core_web_sm)          POS tagging, dependency parsing (nsubj), entity isolation
Synonym Expansion              NLTK (wordnet)                   Generating query variations for news search
News Retrieval                 gnews                            Real-time article headline and snippet aggregation
Fact Ranking                   sentence-transformers            Cross-Encoder for sentence relevance scoring
                               (ms-marco-MiniLM-L-6-v2)
Stance Classification          transformers                     NLI logical entailment vs. contradiction evaluation
                               (FacebookAI/roberta-large-mnli)


Contributing

Pull requests are warmly welcome. If you'd like to improve news scraping capabilities (e.g., full-text scraping via BeautifulSoup) or extend the hedge word dictionary for different domains (politics, finance), please fork the repository and open a pull request.


License

Distributed under the MIT License. See LICENSE for details.

```
