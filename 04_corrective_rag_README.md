# 04 🛠️ Corrective RAG (CRAG) Pipeline (Mock Implementation)

This repository contains a simplified, fully offline simulation of a Corrective RAG (Corrective Retrieval-Augmented Generation) system.

Unlike a standard RAG system—which trusts its initial database search blindly—a CRAG system adds a critical quality-control step. It grades the retrieved documents for relevance and automatically changes its strategy if the search results are poor, ambiguous, or completely off-topic.

## 🧠 The Core Components
The system relies on a set of automated components to handle the retrieval and grading pipeline:

- **The Retriever:** Performs an initial semantic search across the internal document store (`DOCUMENTS`) using a vector embedder to find the most relevant context for the query.
- **The Evaluator/Grader:** Acts as a quality-control filter. It checks the keyword overlap between your question and the retrieved documents, assigning a confidence percentage and a Verdict (**CORRECT**, **AMBIGUOUS**, or **INCORRECT**).
- **The Query Rewriter:** If a query is vague or ambiguous, this component optimizes and expands the prompt with better context and keywords to try a second, more effective search.
- **The Web Search Fallback:** If the internal database doesn't have the answer, this component serves as a backup, querying a mock web index (`WEB_RESULTS`) for up-to-date internet knowledge.

## 🔀 The Branching Logic (The Decision Tree)
Once the Grader evaluates the initial retrieval, the pipeline splits into three distinct branches:

| Verdict | Condition | Action Taken |
|---------|-----------|--------------|
| 🟢 **CORRECT** | High relevance overlap ($\ge$ 45%) | Passes the retrieved documents straight to the LLM to generate the answer. |
| 🟡 **AMBIGUOUS** | Moderate relevance overlap (20% - 44%) | Sends the query to the Query Rewriter, triggers a re-retrieval with the new query, and sends the fresh results to the LLM. |
| 🔴 **INCORRECT** | Low/No relevance overlap (< 20%) | Discards the useless internal documents entirely and triggers the Web Search Fallback to gather correct context. |

## 🔄 The Execution Process
When a query is processed by the pipeline, it runs through 4 clean steps:

1. **Initial Retrieval:** The system pulls the top 3 closest document matches from the internal database.
2. **Evaluation:** The Grader assesses the documents and issues a Verdict.
3. **Branching & Correction:** The system dynamically adjusts based on the verdict (Proceeds normally, Rewrites & Re-searches, or Falls back to Web Search).
4. **Generation:** The final context is handed off to the LLM along with the original question to print a well-informed answer.
