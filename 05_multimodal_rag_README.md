# 05 🖼️ Multimodal RAG Pipeline (Mock Implementation)

This repository contains a simplified, fully offline simulation of a Multimodal RAG (Retrieval-Augmented Generation) system.

Unlike standard RAG systems that only understand plain text chunks, a Multimodal RAG system treats text, image captions, charts, and structured tables as equals. It maps them all into a single mathematical neighborhood, retrieves a mixed-media bundle of relevant evidence, and passes it to a Multimodal LLM (Vision-Language Model) to generate an answer.

## 🧠 The Architecture
The system is engineered around a cross-modal strategy to handle diverse data types:

- **Shared Embedding Space:** Standard text, image metadata (captions/tags), and structural table strings are converted into vectors using a single unified encoder layout (`MultimodalEmbedder`). Modality-specific offsets align disparate mediums into a matching 64-dimensional vector universe.
- **Unified Vector Index:** Rather than hosting separate databases for text and media files, all processed vectors are saved inside a single, queryable `MultimodalVectorIndex`.
- **Modality Preference Boosting:** Before querying the index, the pipeline scans the user's phrasing for visual intent indicators (e.g., words like "chart", "diagram", or "table"). If detected, matching mediums receive an automatic relevance score bump to prioritize rich media layouts in the final context payload.
- **Multimodal LLM:** Represents the generation engine capable of reading text descriptions, chart plots, and data tables concurrently to form a comprehensive answer.

## 📦 Data Mediums (The Corpus)
The script initializes three different mock knowledge pools:

- `TEXT_CORPUS`: Clean text strings outlining historical releases and model behaviors.
- `IMAGE_CORPUS`: Nested image dictionaries packing file paths, diagnostic tags, and captions describing chart visuals (e.g., benchmark curves, architectural blueprints).
- `TABLE_CORPUS`: Structural tabular data holding exact evaluation headers, model rows, and benchmark percentages.

## 🔄 The 5-Step Execution Flow
When a hybrid query is submitted—such as *"How do GPT-4 and Claude 3 compare on benchmarks? Show me charts or tables."*—the system executes the following steps:

1. **Index Construction:** Maps all raw text, images, and tables into the shared vector index.
2. **Preference Tagging:** Analyzes the question's text keywords to determine if the user prefers graphical charts, raw tables, or general prose.
3. **Unified Cross-Modal Retrieval:** Searches the shared vector space to extract the top $K$ most similar items, regardless of their native media type.
4. **Relevance Score Boosting:** Multiplies the raw similarity scores of preferred media items by a boosting weight (e.g., +15% for requested charts) to surface layout-heavy content.
5. **Multimodal Synthesis:** Compiles the final mixed text, table grids, and image markers into a rich context package for the Multimodal LLM to read and formulate the answer.

## 💻 Example Output
When running the script, the terminal will show a log of this process in action:

```text
============================================================
MULTIMODAL RAG PIPELINE
============================================================
Query: How do GPT-4 and Claude 3 compare on benchmarks? Show me charts or tables. 

Step 1 — Unified Index built:
  4 text chunks | 3 images | 2 tables
  Total: 9 documents indexed in shared 64-dim space

Step 2 — Query Modality Preferences:
  Wants images/charts: True
  Wants tables:        True

Step 3 — Unified Retrieval (top 5 across all modalities):
  [IMAGE] score=0.202  Image: LLM scaling laws plot
  [IMAGE] score=0.156  Image: Benchmark comparison bar chart
  [TEXT ] score=0.154  Text: Gemini Ultra benchmark description
  [TEXT ] score=0.133  Text: GPT-4 release description
  [TABLE] score=0.096  Table: LLM model comparison with parameters and context length

Step 4 — After Modality Boosting (top 5):
  [IMAGE] score=0.232  Image: LLM scaling laws plot
  [IMAGE] score=0.179  Image: Benchmark comparison bar chart
  [TEXT ] score=0.154  Text: Gemini Ultra benchmark description
  [TEXT ] score=0.133  Text: GPT-4 release description
  [TABLE] score=0.106  Table: LLM model comparison with parameters and context length

Step 5 — Multimodal LLM Answer:
[Mock Multimodal LLM Answer]
Query: How do GPT-4 and Claude 3 compare on benchmarks? Show me charts or tables. 

Retrieved Context (5 items — text=2, images=2, tables=1):
  (score=0.232) [IMAGE] llm_scaling_laws.png — Log-log plot showing LLM performance scaling with compute and parameter count
  (score=0.179) [IMAGE] benchmark_comparison_chart.png — Bar chart comparing GPT-4, Claude 3, Gemini on MMLU, HumanEval, and GSM8K benchmarks
  (score=0.154) [TEXT] Google Gemini Ultra achieves state-of-the-art results on MMLU benchmark, surpassing human experts.
  (score=0.133) [TEXT] OpenAI GPT-4 was released in March 2023 with multimodal capabilities including image understanding.
  (score=0.106) [TABLE] LLM Model Comparison
  Model | Company | Params (B) | Context (K) | Year
  GPT-4 | OpenAI | ~1800 | 128 | 2023
  Claude 3 Opus | Anthropic | ~200 | 200 | 2024
  Gemini Ultra | Google | ~1000 | 1000 | 2024
  LLaMA 3 70B | Meta | 70 | 8 | 2024
  Mistral 7B | Mistral | 7 | 32 | 2023

Answer: Based on the benchmark comparison table and chart, Claude 3 Opus leads on MMLU (86.8%) and HumanEval (84.9%), while GPT-4 scores 86.4% on MMLU and 67.0% on HumanEval. The bar chart visually confirms Claude 3 Opus outperforms GPT-4 on coding benchmarks. Gemini Ultra achieves the highest MMLU score (90.0%).
============================================================
```
