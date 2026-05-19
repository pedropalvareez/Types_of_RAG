# 02 The Big Picture Flow

Instead of looking at your documents as isolated sentences, the code converts them into a social network of tech companies, people, and tools, then navigates that network to answer your question.

## Step-by-Step Code Breakdown

### Step 1: Building the Knowledge Graph (`build_knowledge_graph`)
The code scans the list of raw sentences (`DOCUMENTS`) and extracts Entities (nouns like Sam Altman or OpenAI) and Relations (verbs like `ceo_of` or `founded`).
- Using a Python library called `networkx`, it maps these out as a web of connected points.
- **Result:** A structured graph database is built completely offline.

### Step 2: Query Entity Extraction (`extract_entities`)
You ask the system: *"What is the relationship between Sam Altman and Anthropic?"*
- The script runs a quick check against its known list and extracts the core subjects you are asking about.
- **Result:** It flags Sam Altman and Anthropic as the target entry points.

### Step 3: Subgraph Retrieval (`retrieve_subgraph`)
If the system only looked at a sentence containing "Sam Altman" and a sentence containing "Anthropic", it would miss the connection because no single sentence contains both names.
- To fix this, the code grabs the target nodes and plays a game of "degrees of separation" (set to 2 hops). It grabs Sam Altman, Anthropic, and any entity directly connected to either of them.
- **Result:** It isolates a tiny, relevant cluster (a subgraph) out of the massive web. Both paths naturally collide at OpenAI.

### Step 4: Community Summary (`summarize_subgraph`)
The code translates that isolated visual cluster back into plain, structured text that an AI can read easily. It outputs a neat list of links:
```text
[Person] Sam Altman --ceo_of--> [Company] OpenAI
[Person] Dario Amodei --left--> [Company] OpenAI
[Person] Dario Amodei --founded--> [Company] Anthropic
```

### Step 5: The LLM Answer (`mock_llm`)
Finally, the query and that clean list of links are handed over to the Language Model. Because the graph connected the dots for it, the LLM doesn't have to guess. It instantly sees the bridge:
> **Answer:** Sam Altman is the CEO of OpenAI. Dario and Daniela Amodei left OpenAI to found Anthropic. Therefore, they share a historical connection through OpenAI.

## Why this approach is smart
If you used standard vector search (Traditional RAG), a computer might look for the words "Sam Altman" and "Anthropic" together, fail to find a direct sentence match, and give you a weak answer.
By mapping the data into a Knowledge Graph first, the system discovers indirect relationships (Person A &rarr; Company B &rarr; Person C) effortlessly.

## 💻 Example Output
```text
============================================================
GRAPH RAG PIPELINE
============================================================
Query: What is the relationship between Sam Altman and Anthropic?

Step 1 — Knowledge Graph built: 18 nodes, 9 edges
  Nodes: ['Sam Altman', 'OpenAI', 'GPT-4', 'Dario Amodei', 'Daniela Amodei', 'Anthropic', 'Microsoft', 'Azure', 'Bing', 'which'] …

Step 2 — Query entities detected: ['Sam Altman', 'Anthropic']

Step 3 — Subgraph retrieved: 10 nodes, 8 edges

Step 4 — Subgraph / Community Summary:
  [Person] Sam Altman --ceo_of--> [Company] OpenAI
  [Company] OpenAI --developed--> [Technology] GPT-4
  [Company] OpenAI --backed_by--> [Company] Microsoft
  [Person] Dario Amodei --left--> [Company] OpenAI
  [Company] Anthropic --developed--> [Technology] Claude
  [Company] Google --invested_in--> [Company] Anthropic
  [Unknown] weight LLMs and --competes_with--> [Company] OpenAI
  [Company] NVIDIA --supplies_to--> [Company] OpenAI

Step 5 — LLM Answer:
[Mock LLM Answer]
Query: What is the relationship between Sam Altman and Anthropic?

Knowledge Graph Context:
  [Person] Sam Altman --ceo_of--> [Company] OpenAI
  [Company] OpenAI --developed--> [Technology] GPT-4
  [Company] OpenAI --backed_by--> [Company] Microsoft
  [Person] Dario Amodei --left--> [Company] OpenAI
  [Company] Anthropic --developed--> [Technology] Claude
  [Company] Google --invested_in--> [Company] Anthropic
  [Unknown] weight LLMs and --competes_with--> [Company] OpenAI
  [Company] NVIDIA --supplies_to--> [Company] OpenAI

Answer: Sam Altman is the CEO of OpenAI. Dario Amodei and Daniela Amodei left OpenAI to found Anthropic in 2021. So Sam Altman and Anthropic share a historical connection through OpenAI, though he is not affiliated with Anthropic.
============================================================
```
