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
