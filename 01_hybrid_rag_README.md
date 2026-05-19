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
**What the code does:** It uses RRF. Instead of trying to mix completely different math scores, it looks only at the rank (1st place, 2nd place, 3rd place) of the documents in both lists.
It applies a simple formula: $\frac{1}{k + \text{rank}}$. Documents that scored high in both lists quickly rise to the very top of the final list.

### 5. Generation (mock_llm)
In a production app, the top 2 or 3 documents found during the merge would be sent to an LLM (like GPT-4) to write a nice answer.
**What the code does:** Since it's offline, the `mock_llm` function simply prints out the text it retrieved and hardcodes a clean, simulated response matching the data.

## Summary of the Pipeline Flow
```text
User Query ──> [ Dense Vector Search ] ──> List of best matches (by meaning) ──╮
            ──> [ Sparse BM25 Search ] ──> List of best matches (by words)   ──┼─> [ RRF Merge ] ──> Top Chunks ──> [ LLM ] ──> Final Answer
```

When you run this script, it executes all these steps sequentially in your terminal, showing you exactly how the documents shift ranks before being handed off to the final text generator.
