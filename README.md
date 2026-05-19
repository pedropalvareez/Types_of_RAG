# RAG Architecture Patterns

Five self-contained Python examples demonstrating different Retrieval-Augmented
Generation (RAG) architectures. Every script runs fully offline — no API keys
required. All LLM responses and embeddings are mocked or generated with
deterministic random vectors so the focus stays on the pipeline logic.

## Setup

```bash
pip install -r requirements.txt
```

## Patterns

### 1. Hybrid RAG (`01_hybrid_rag.py`)

Combines **dense vector search** (fake CLIP-style embeddings + cosine similarity
store) with **sparse keyword search** (hand-rolled BM25). Both retrievers run in
parallel on the same query, then results are merged via **Reciprocal Rank Fusion
(RRF)**. The fused top-K chunks are passed to the LLM.

```
Query → Dense Retrieval ─┐
      → BM25 Retrieval  ─┴→ RRF → Top-K chunks → LLM → Answer
```

```bash
python 01_hybrid_rag.py
```

---

### 2. GraphRAG (`02_graph_rag.py`)

Builds a **Knowledge Graph** (using `networkx`) from the document corpus.
Named entities (Person, Company, Technology, Location) become nodes; typed
relationships (founded, ceo_of, invested_in, …) become directed edges.
At query time, query entities seed a **multi-hop subgraph traversal**; the
retrieved subgraph is summarised into a community description fed to the LLM.

```
Query → Entity Extraction → KG Subgraph Retrieval → Community Summary → LLM → Answer
```

```bash
python 02_graph_rag.py
```

---

### 3. Agentic RAG (`03_agentic_rag.py`)

A **Planner Agent** analyses the query and selects an ordered sequence of tools
to invoke. Three tools are available: `VectorSearchTool`, `WebSearchTool`, and
`SQLDatabaseTool`. The agent loop executes each tool, tracks accumulated
confidence, and short-circuits when confidence is high enough. A **Reasoner
Agent** then synthesises all tool results into a final answer.

```
Query → Planner → [Tool Loop: Vector / Web / SQL] → Reasoner → Answer
```

```bash
python 03_agentic_rag.py
```

---

### 4. Corrective RAG — CRAG (`04_corrective_rag.py`)

After initial retrieval an **Evaluator/Grader** scores relevance via keyword
overlap and emits one of three verdicts:

- **CORRECT** → pass chunks directly to the LLM.
- **AMBIGUOUS** → rewrite the query and re-retrieve, then go to LLM.
- **INCORRECT** → fall back to a simulated web search, then go to LLM.

Three demo queries exercise all three branches in one run.

```
Query → Retriever → Evaluator ──CORRECT──→ LLM → Answer
                             ├─AMBIGUOUS─→ Rewriter → Re-retrieve → LLM → Answer
                             └─INCORRECT─→ Web Search → LLM → Answer
```

```bash
python 04_corrective_rag.py
```

----

### 5. Multimodal RAG (`05_multimodal_rag.py`)

Indexes three modality types — **text chunks**, **images/charts** (represented
by captions and tags), and **tables** — into a single unified vector index using
a mock CLIP-style embedder that maps every modality to the same 64-dimensional
space. At query time, modality preferences are detected ("chart", "table", …)
and a post-retrieval **modality boost** re-ranks results before the mock
Multimodal LLM generates an answer referencing text, images, and tables together.

```
Text Chunks ─┐
Images      ─┼→ Multimodal Embedder → Unified Index → Retrieval → Boost → MLLM → Answer
Tables      ─┘
```

```bash
python 05_multimodal_rag.py
```

---

## Dependencies

| Library    | Used by                          |
|------------|----------------------------------|
| `numpy`    | all files (fake embeddings)      |
| `networkx` | `02_graph_rag.py` (KG traversal) |

BM25 is implemented from scratch in `01_hybrid_rag.py` — no extra package needed.

# Types_of_RAG
