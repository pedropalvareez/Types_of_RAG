# 01 The Core Concept: Why Hybrid? 

If you ask an AI a question, it needs to look up facts in a database first. There are two classic ways to search, and this script combines them because they complement each other perfectly:

- **Dense Search (Semantic):** Understands the meaning and context of your words (e.g., understanding that "feline" and "cat" are related).
- **Sparse Search (Keyword):** Looks for exact word matches (e.g., finding the exact serial number or unique name like "H100").

## Step-by-Step Code Breakdown

### 1. The Knowledge Base (DOCUMENTS & QUERY)
The script sets up a mini-database of 8 facts about tech companies. The user asks:
*"Who founded OpenAI and what models do they make?"*

### 2. Dense Vector Search (FakeEmbedder & VectorStore)
In a real system, an embedding model turns text into a long string of numbers (vectors) that represent its meaning.
**What the code does:** Because it runs offline, `FakeEmbedder` uses a clever math trick (hashing the text) to generate predictable, fake number strings for each document.
**The Search:** `VectorStore` uses Cosine Similarity (a math formula that measures the angle between two lines) to see which document's numbers look closest to the query's numbers.

### 3. Sparse Keyword Search (BM25)
BM25 is a classic search engine algorithm (the older cousin of Google's search algorithms).
**What the code does:** It counts how many times the words in your query ("OpenAI", "founded", "models") appear in each document. It also penalizes words that are too common (like "and" or "what") and adjusts for document length so short, punchy matches score higher.

### 4. The Merge: Reciprocal Rank Fusion (reciprocal_rank_fusion)
Now you have two separate lists of results. Dense search might think Document A is best, while Sparse search thinks Document B is best. How do you decide?

Because a Cosine Similarity score might be `0.08` and a BM25 score might be `3.38`, you can't just add them together. Instead, the script uses Reciprocal Rank Fusion (RRF).

- It completely ignores the raw mathematical scores from Steps 1 and 2 and looks **only at the rank** (1st place, 2nd place, 3rd place, etc.).
- It gives each document a new score based on this formula: `1 / (60 + rank)`
- **Example:** If Document A placed **1st** in Dense Search and **3rd** in Sparse Search, its final RRF score is: `(1 / 61) + (1 / 63) = 0.0322`.

This fusion method ensures that documents that score decently well across *both* meaning and keywords rise to the very top!

### 5. Generation (mock_llm)
In a production app, the top 2 or 3 documents found during the merge would be sent to an LLM (like GPT-4) to write a nice answer.
**What the code does:** Since it's offline, the `mock_llm` function simply prints out the text it retrieved and hardcodes a clean, simulated response matching the data.

## Summary of the Pipeline Flow
```text
User Query ──> [ Dense Vector Search ] ──> List of best matches (by meaning) ──╮
            ──> [ Sparse BM25 Search ] ──> List of best matches (by words)   ──┼─> [ RRF Merge ] ──> Top Chunks ──> [ LLM ] ──> Final Answer
```

When you run this script, it executes all these steps sequentially in your terminal, showing you exactly how the documents shift ranks before being handed off to the final text generator.

## 💻 Example Output
```text
============================================================
HYBRID RAG PIPELINE
============================================================
Query: Who founded OpenAI and what models do they make?

Step 1 — Dense (vector) retrieval:
  doc_id=1  score=0.0857  text=Google DeepMind created AlphaGo, the first AI to defeat a wo…
  doc_id=5  score=0.0514  text=Hugging Face hosts thousands of open-source machine learning…
  doc_id=7  score=0.0445  text=Mistral AI is a Paris-based startup that released high-perfo…
  doc_id=0  score=-0.0131  text=OpenAI was founded in 2015 by Sam Altman and Elon Musk. It d…
  doc_id=2  score=-0.0172  text=Anthropic was founded in 2021 by former OpenAI researchers D…

Step 2 — Sparse (BM25) retrieval:
  doc_id=0  score=3.3812  text=OpenAI was founded in 2015 by Sam Altman and Elon Musk. It d…
  doc_id=2  score=3.1321  text=Anthropic was founded in 2021 by former OpenAI researchers D…
  doc_id=3  score=2.5962  text=Microsoft invested heavily in OpenAI and integrated GPT mode…
  doc_id=5  score=1.4454  text=Hugging Face hosts thousands of open-source machine learning…
  doc_id=4  score=0.7227  text=Meta AI released the LLaMA family of open-weight language mo…

Step 3 — Reciprocal Rank Fusion (merged ranking):
  doc_id=0  rrf_score=0.0320  text=OpenAI was founded in 2015 by Sam Altman and Elon Musk. It d…
  doc_id=5  rrf_score=0.0318  text=Hugging Face hosts thousands of open-source machine learning…
  doc_id=2  rrf_score=0.0315  text=Anthropic was founded in 2021 by former OpenAI researchers D…

Step 4 — LLM Answer:
[Mock LLM Answer]
Query: Who founded OpenAI and what models do they make?

Based on the retrieved context:
- OpenAI was founded in 2015 by Sam Altman and Elon Musk. It develops large language models including GPT-4.
- Hugging Face hosts thousands of open-source machine learning models and datasets on its platform.
- Anthropic was founded in 2021 by former OpenAI researchers Dario Amodei and Daniela Amodei.

Answer: OpenAI was co-founded by Sam Altman and Elon Musk in 2015. They are known for developing GPT-4 and other large language models.
============================================================
```
