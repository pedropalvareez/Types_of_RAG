# 03 Agentic RAG Pipeline (Mock Implementation)

This repository contains a simplified simulation of an Agentic RAG (Retrieval-Augmented Generation) system.

Unlike a standard RAG system—which blindly searches a single database and summarizes the results—an Agentic RAG system acts like a human researcher. It evaluates the prompt, formulates a research plan, selects the right tools for the job, and synthesizes the final answer.

## 🧠 The Architecture
The system is powered by two main "Agents" that manage the workflow:

- **The Planner Agent (The Manager):** Analyzes the user's query and decides which tools are needed. For example, if it sees the year "2024," it knows to query the SQL database. If it sees the word "compare," it triggers a web search. It creates a step-by-step execution plan.
- **The Reasoner Agent (The Writer):** Once the data is gathered, this agent reads through the raw results, synthesizes the context, and writes a clean, accurate, human-readable final answer.

## 🛠️ The Tools & Data Stores
Before the AI can answer anything, it needs access to information. This script uses three specific tools to search three different types of mock data:

- **VectorSearchTool:** Searches `VECTOR_DOCS` (unstructured text sentences). It uses semantic search to find the underlying meaning and context of a query.
- **SQLDatabaseTool:** Searches `SQL_TABLE` (a structured database). It filters hard facts based on exact numbers and keywords (e.g., model name, company, size, release year).
- **WebSearchTool:** Searches `WEB_INDEX` (a simulated internet index). It looks for keyword matches to find the latest news, comparisons, or benchmarks.

## 🔄 The Execution Loop
When you run a query like *"Which AI models were released in 2024, and how do they compare on benchmarks?"*, the system runs through a 4-step loop:

1. **Planning:** The Planner Agent maps the query to the available tools and creates a step-by-step plan.
2. **Executing:** The system runs each tool on the plan sequentially, collecting data and returning a "Confidence Score" (0.0 to 1.0) for each result.
3. **Evaluating (Early Stopping):** After each tool runs, the system checks the average confidence score. If the gathered evidence is strong enough (e.g., > 80% confidence), it stops searching early to save time and computing power.
4. **Answering:** All gathered evidence is passed to the Reasoner Agent, which formats the final response for the user.

## 💻 Example Output
When running the script, the terminal will show a log of this process in action:

```text
============================================================
AGENTIC RAG PIPELINE
============================================================
Query: Which AI models were released in 2024, and how do they compare on benchmarks?

Step 1 — Planner selected 3 tool(s):
  → vector_search: 'Which AI models were released in 2024, and how do they compa'
  → sql_database: 'Which AI models were released in 2024, and how do they compa'
  → web_search: 'benchmark comparison LLMs'

Step 2.1 — vector_search results (confidence=0.65):
  • Meta's LLaMA 3 supports 128k context windows and runs on consumer hardware.   
  • OpenAI released GPT-4 in March 2023, showing significant capability improvements.
  • Anthropic's Claude 3 Opus achieved top scores on several reasoning benchmarks.

Step 2.2 — sql_database results (confidence=0.85):
  • Claude 3 Opus by Anthropic (200B params, 2024)
  • Gemini Ultra by Google (1000B params, 2024)
  • LLaMA 3 70B by Meta (70B params, 2024)

Step 2.3 — web_search results (confidence=0.75):
  • Claude 3 Opus leads on MMLU; GPT-4 leads on HumanEval; Gemini Ultra leads on multimodal tasks.

Step 3 — Reasoner Agent:
[Mock Reasoner Agent Answer]
Query: Which AI models were released in 2024, and how do they compare on benchmarks?

Aggregated Tool Evidence:
[VECTOR_SEARCH — confidence=0.65]
  • Meta's LLaMA 3 supports 128k context windows and runs on consumer hardware.   
  • OpenAI released GPT-4 in March 2023, showing significant capability improvements.
  • Anthropic's Claude 3 Opus achieved top scores on several reasoning benchmarks.
[SQL_DATABASE — confidence=0.85]
  • Claude 3 Opus by Anthropic (200B params, 2024)
  • Gemini Ultra by Google (1000B params, 2024)
  • LLaMA 3 70B by Meta (70B params, 2024)
[WEB_SEARCH — confidence=0.75]
  • Claude 3 Opus leads on MMLU; GPT-4 leads on HumanEval; Gemini Ultra leads on multimodal tasks.

Final Answer: In 2024, notable AI models include Claude 3 Opus (Anthropic), Gemini Ultra (Google), and LLaMA 3 (Meta). On benchmarks, Claude 3 Opus leads on MMLU, GPT-4 leads on HumanEval, and Gemini Ultra excels at multimodal tasks.
```
