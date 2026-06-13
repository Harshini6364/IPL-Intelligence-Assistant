# 🏏 IPL Intelligence Assistant

> A Multi-Agent RAG system built with LangGraph, Groq, and ChromaDB that answers IPL cricket queries by routing them through specialised AI agents — batting, bowling, venue, head-to-head, form, and more.

![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat&logo=python)
![LangGraph](https://img.shields.io/badge/LangGraph-0.2+-green?style=flat)
![Groq](https://img.shields.io/badge/LLM-Groq%20Llama%203.1-orange?style=flat)
![ChromaDB](https://img.shields.io/badge/VectorDB-ChromaDB-purple?style=flat)
![Streamlit](https://img.shields.io/badge/UI-Streamlit-red?style=flat)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat)

---

## 📖 Table of Contents

- [About the Project](#about-the-project)
- [Why This Project?](#why-this-project)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Installation and Setup](#installation-and-setup)
- [How to Use](#how-to-use)
- [LangGraph Architecture](#langgraph-architecture)
- [Sample Queries](#sample-queries)
- [Credits](#credits)
- [License](#license)

---

## 📌 About the Project

The **IPL Intelligence Assistant** is a Retrieval-Augmented Generation (RAG) application that answers questions about IPL cricket using a **multi-agent LangGraph workflow**.

Unlike a simple single-chain RAG pipeline, this system:

- **Routes** each query to the most relevant agent node (batting stats, bowling stats, venue reports, head-to-head records, recent form, IPL milestones)
- **Chains multiple agents** together for complex queries like Dream11 team suggestions and match predictions
- **Detects conflicts** in data using a dedicated ValidationNode
- **Displays the full node trace** so you can see exactly which agents activated for each query

Built as part of **RAG-ATHON 24 — Advanced Track** at SVECW, Department of Information Technology.

---

## 💡 Why This Project?

Standard RAG pipelines use a single chain: `Query → Retriever → LLM → Answer`. This works for simple lookups but fails when a query needs multiple data sources — for example, a Dream11 recommendation needs player form data, batting stats, bowling stats, and venue conditions all combined.

**LangGraph solves this** by modelling the pipeline as a stateful directed graph where:
- Nodes are specialised agents
- Edges are conditional routing decisions
- A shared State object carries information between all nodes

This project demonstrates that architecture on real IPL data.

---

## 🛠️ Tech Stack

| Layer | Tool | Reason |
|---|---|---|
| Agent Orchestration | LangGraph 0.2+ | Stateful multi-agent graph |
| LLM | Groq — Llama 3.1 8B Instant | Fast, free inference |
| Vector Store | ChromaDB | Local persistent embeddings |
| Embeddings | HuggingFace all-MiniLM-L6-v2 | Lightweight, accurate |
| PDF Loading | PyPDF + LangChain | Structured document ingestion |
| UI | Streamlit | Rapid interactive interface |
| Language | Python 3.10+ | |

---

## 📁 Project Structure

```
ipl-langgraph-rag/
├── .env                        # API keys — never commit this
├── .env.example                # Safe template for reviewers
├── .gitignore
├── requirements.txt
├── setup_venv.sh               # One-command environment setup
├── main.py                     # Terminal test runner
├── app.py                      # Streamlit UI
├── data/
│   └── IPL_LangGraph_RAG_Dataset.pdf
├── graph/
│   ├── __init__.py
│   ├── state.py                # IPLAgentState TypedDict
│   ├── nodes.py                # All core agent nodes
│   ├── team_node.py            # TeamProfileNode
│   ├── validation.py           # ValidationNode
│   └── graph_builder.py        # Full graph wiring
└── rag/
    ├── __init__.py
    ├── ingest.py               # PDF → chunks → ChromaDB
    └── retriever.py            # Metadata-filtered retrieval
```

---

## ⚙️ Installation and Setup

### Prerequisites

- Python 3.10 or higher
- A free Groq API key from [console.groq.com](https://console.groq.com)

### Step 1 — Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/ipl-langgraph-rag.git
cd ipl-langgraph-rag
```

### Step 2 — Create virtual environment

```bash
bash setup_venv.sh
```

This creates `venv/` and installs all packages from `requirements.txt` automatically.

**Activate manually in future sessions:**

```bash
# Mac / Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### Step 3 — Set your API key

Copy `.env.example` to `.env` and add your Groq key:

```bash
cp .env.example .env
```

Open `.env` and replace the placeholder:

```
GROQ_API_KEY=your_actual_key_here
```

### Step 4 — Add the dataset

Place `IPL_LangGraph_RAG_Dataset.pdf` inside the `data/` folder.

### Step 5 — Run

```bash
# Terminal test — builds vector store on first run (~2 minutes)
python main.py

# Streamlit UI
streamlit run app.py
```

The first run downloads the embedding model and builds the ChromaDB vector store. Subsequent runs are instant.

---

## 🚀 How to Use

### Terminal

Running `python main.py` automatically tests 8 sample queries and prints:
- The query type detected by RouterNode
- The chain of nodes activated
- The final answer (first 300 characters)

### Streamlit UI

After running `streamlit run app.py`, open `http://localhost:8501` in your browser.

- Type any IPL query in the text box, **or**
- Click a sample query from the sidebar
- The answer appears along with:
  - Query type detected
  - Entities extracted (team/player names)
  - Full node activation chain
  - Retrieved context chunks (expandable)

**Screenshot — Dream11 query showing multi-node activation:**

```
Nodes activated: RouterNode → FormNode → BattingStatsNode → BowlingStatsNode → VenueNode → SynthesisNode → ValidationNode
```

---

## 🧠 LangGraph Architecture

### State Schema

Every node reads from and writes to a shared `IPLAgentState` TypedDict. This is the core LangGraph concept — state flows through the graph and accumulates context at each node.

### Node Routing Map

| Node | Triggered When | Retrieves |
|---|---|---|
| RouterNode | Every query | Classifies type, extracts entities |
| TeamProfileNode | "captain", "coach", "titles" | Team table + season history |
| BattingStatsNode | "runs", "average", "strike rate" | Batting stats rows |
| BowlingStatsNode | "wickets", "economy", "figures" | Bowling stats rows |
| VenueNode | "pitch", "stadium", "dew" | Venue narrative reports |
| H2HNode | "vs", "head-to-head", prediction | Head-to-head records |
| FormNode | "recent", "last 5", dream11 | Last 5 match scores |
| RecordsNode | "highest", "most", "fastest" | IPL milestones table |
| SynthesisNode | Always last | Calls Groq LLM with all context |
| ValidationNode | Always after Synthesis | Detects conflicting data values |

### Multi-Node Paths

**Prediction query** (`who will win`):
```
Router → H2HNode → VenueNode → FormNode → BattingNode → BowlingNode → Synthesis → Validation
```

**Dream11 query** (`suggest XI`):
```
Router → FormNode → BattingNode → BowlingNode → Synthesis → Validation
```

**Simple query** (`who captains CSK`):
```
Router → TeamProfileNode → Synthesis → Validation
```

---

## 💬 Sample Queries

| Difficulty | Query |
|---|---|
| Easy | Who captains Chennai Super Kings in 2024? |
| Easy | What is Virat Kohli's career IPL run tally? |
| Easy | What is the highest team total in IPL history? |
| Medium | List all bowlers with an economy rate below 7.0 |
| Medium | Which opener has the highest strike rate? |
| Medium | How many times have MI and CSK played each other? |
| Hard | Suggest a Dream11 XI for MI vs SRH at Wankhede tonight |
| Hard | CSK is playing RCB at Chinnaswamy. Who will win? |
| Hard | What bowling strategy should SRH use at MA Chidambaram? |
| Expert | Is Yuzvendra Chahal's wicket count 205 or 187? |

---

🚀 Future Improvements

Short Term


Streaming responses — stream LLM output token by token instead of waiting for full answer, just like ChatGPT
Conversation export — download full chat history as PDF or CSV
Named sessions — save and switch between multiple chat sessions like ChatGPT's conversation list
Voice input — allow users to ask questions by speaking using browser speech API


Medium Term


Live IPL data — integrate Cricbuzz or ESPN Cricinfo API to fetch real-time scores, updated squad data, and live match conditions instead of relying on a static PDF
Web search fallback — when a query falls outside the dataset (e.g. IPL 2025 auction prices), automatically trigger a web search using Tavily or SerpAPI
Reranking — add a Cross-Encoder reranker after retrieval to improve chunk quality before sending to the LLM
Query expansion — automatically rephrase ambiguous queries before retrieval to improve recall


Long Term


User authentication — login system so each user has their own private history and preferences
Dream11 accuracy scoring — compare AI Dream11 suggestions against actual match results and score prediction accuracy over time
Multi-language support — support queries in Hindi and Telugu for wider accessibility
Mobile app — wrap the Streamlit app into a mobile-friendly PWA or build a React Native version
Fine-tuned model — fine-tune a small LLM specifically on IPL data for faster, more accurate answers without needing RAG


---

## 🙏 Credits

- **Dataset** — IPL LangGraph RAG Dataset, RAG-ATHON 24, SVECW Department of Information Technology
- **LangGraph Docs** — [python.langchain.com/docs/langgraph](https://python.langchain.com/docs/langgraph)
- **Groq** — [console.groq.com](https://console.groq.com) for free LLM inference
- **ChromaDB** — [trychroma.com](https://trychroma.com)
- **README structure guide** — [FreeCodeCamp](https://www.freecodecamp.org/news/how-to-write-a-good-readme-file/)

---

## 📄 License

This project is licensed under the **MIT License** — you are free to use, modify, and distribute this code with attribution.

```
MIT License

Copyright (c) 2024

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```
