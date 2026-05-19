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

## 💻 Example Output
```text
============================================================
CORRECTIVE RAG (CRAG) PIPELINE
============================================================
Query: Who founded Anthropic and what do they build?

Step 1 — Initial Retrieval:
  [doc 4] GPT-4 was released by OpenAI in March 2023 and supports multimodal inp… 
  [doc 0] Anthropic was founded in 2021 by Dario Amodei and Daniela Amodei.…      
  [doc 6] Retrieval-Augmented Generation (RAG) combines retrieval with generatio… 

Step 2 — Evaluator Verdict: AMBIGUOUS  (Moderate overlap (38%) — may need rewriting)

Step 3 — Branch: AMBIGUOUS → rewrite query and re-retrieve
  Rewritten query: 'anthropic company founders history'
  Re-retrieved docs:
    [doc 1] Claude is Anthropic's AI assistant, designed with a focus on safety.… 
    [doc 4] GPT-4 was released by OpenAI in March 2023 and supports multimodal inp…
    [doc 6] Retrieval-Augmented Generation (RAG) combines retrieval with generatio…

Step 4 — Final Answer:
[Mock LLM Answer — source: re-retrieval after rewrite]
Query: Who founded Anthropic and what do they build?

Context used:
  [1] Claude is Anthropic's AI assistant, designed with a focus on safety.        
  [2] GPT-4 was released by OpenAI in March 2023 and supports multimodal inputs.  
  [3] Retrieval-Augmented Generation (RAG) combines retrieval with generation for better accuracy.

Answer: Based on the available context, Anthropic was founded in 2021 by Dario Amodei and Daniela Amodei. Their flagship product, Claude, is an AI assistant built with safety as a core design principle.
============================================================

============================================================
CORRECTIVE RAG (CRAG) PIPELINE
============================================================
Query: tell me about claude

Step 1 — Initial Retrieval:
  [doc 5] LangChain is a framework for building applications powered by language… 
  [doc 2] The Amazon rainforest covers approximately 5.5 million square kilometr… 
  [doc 0] Anthropic was founded in 2021 by Dario Amodei and Daniela Amodei.…      

Step 2 — Evaluator Verdict: INCORRECT  (Low overlap (0%) — retrieved docs are off-topic)

Step 3 — Branch: INCORRECT → web search fallback
  Web search results:
    • Claude 3 supports 200k token context windows and excels at analysis and coding tasks.

Step 4 — Final Answer:
[Mock LLM Answer — source: web search fallback]
Query: tell me about claude

Context used:
  [1] Claude 3 supports 200k token context windows and excels at analysis and coding tasks.

Answer: Based on the available context, Anthropic was founded in 2021 by Dario Amodei and Daniela Amodei. Their flagship product, Claude, is an AI assistant built with safety as a core design principle.
============================================================

============================================================
CORRECTIVE RAG (CRAG) PIPELINE
============================================================
Query: What is the capital of France and its population?

Step 1 — Initial Retrieval:
  [doc 5] LangChain is a framework for building applications powered by language… 
  [doc 2] The Amazon rainforest covers approximately 5.5 million square kilometr… 
  [doc 4] GPT-4 was released by OpenAI in March 2023 and supports multimodal inp… 

Step 2 — Evaluator Verdict: AMBIGUOUS  (Moderate overlap (33%) — may need rewriting)

Step 3 — Branch: AMBIGUOUS → rewrite query and re-retrieve
  Rewritten query: 'What is the capital of France and its population? details overview background'
  Re-retrieved docs:
    [doc 4] GPT-4 was released by OpenAI in March 2023 and supports multimodal inp…
    [doc 2] The Amazon rainforest covers approximately 5.5 million square kilometr…
    [doc 3] Python is a high-level programming language known for its readability.…

Step 4 — Final Answer:
[Mock LLM Answer — source: re-retrieval after rewrite]
Query: What is the capital of France and its population?

Context used:
  [1] GPT-4 was released by OpenAI in March 2023 and supports multimodal inputs.  
  [2] The Amazon rainforest covers approximately 5.5 million square kilometres.   
  [3] Python is a high-level programming language known for its readability.      

Answer: Based on the available context, Anthropic was founded in 2021 by Dario Amodei and Daniela Amodei. Their flagship product, Claude, is an AI assistant built with safety as a core design principle.
============================================================
```
