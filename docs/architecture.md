# Architecture

This document explains how the Community FAQ Bot is structured,
how data flows through the system, and what each component does.

---

## High Level Overview

The system has two main layers:

**Layer 1 — Knowledge Base (built once, updated when FAQ changes)**
Your FAQ documents get processed, broken into chunks, converted into
vectors, and stored in ChromaDB. This happens upfront and only needs
to run again when the knowledge base is updated.

**Layer 2 — Agent Pipeline (runs on every user question)**
When a user asks a question, a team of AI agents works together to
retrieve the right information and produce an accurate answer.

---

## System Diagram

    ┌─────────────────────────────────────────────────┐
    │                  LAYER 1                        │
    │              Knowledge Base                     │
    │                                                 │
    │  FAQ Documents → Chunker → Embedding Model      │
    │                                    ↓            │
    │                               ChromaDB          │
    └─────────────────────────────────────────────────┘
                              ↑
                         retrieves
                              ↑
    ┌─────────────────────────────────────────────────┐
    │                  LAYER 2                        │
    │               Agent Pipeline                   │
    │                                                 │
    │  User Input                                     │
    │      ↓                                          │
    │  Language Detector Agent                        │
    │      ↓                                          │
    │  Retriever Agent ←→ ChromaDB                    │
    │      ↓                                          │
    │  Answer Agent                                   │
    │      ↓                                          │
    │  Reviewer Agent                                 │
    │      ↓                                          │
    │  Response → User                                │
    └─────────────────────────────────────────────────┘

---

## Layer 1 — Knowledge Base Pipeline

### What it does
Converts human-readable FAQ documents into a searchable vector database.

### The process

**Step 1 — Documents**
FAQ content is written as simple markdown files stored in `data/knowledge_base/`.
Each file covers a topic: pricing, room specs, booking rules, cancellation policy etc.

**Step 2 — Chunking**
Documents are split into small overlapping chunks (~500 words each).
Why overlap? So context isn't lost at chunk boundaries.
Example: a sentence split across two chunks still makes sense in both.

**Step 3 — Embedding**
Each chunk is converted into a vector — a list of numbers that captures
its meaning mathematically. Similar meanings produce similar vectors.
This is what makes semantic search possible.

**Step 4 — Storage**
Vectors and their source text are stored in ChromaDB, a local vector database.
ChromaDB runs entirely on your machine — no external service needed.

### When does this run?
- Once when setting up the project
- Again whenever FAQ documents are updated
- Script: `src/rag/ingest.py`

---

## Layer 2 — Agent Pipeline

### The agents and their jobs

**Language Detector Agent**
- Input: raw user question
- Job: identify whether the question is in English or German
- Output: detected language + original question
- Why needed: so the final answer is returned in the correct language

**Retriever Agent**
- Input: user question + detected language
- Job: search ChromaDB for the most relevant FAQ chunks
- Output: top 3–5 relevant chunks with their source documents
- Why needed: grounds the answer in real content, prevents hallucination

**Answer Agent**
- Input: user question + retrieved chunks
- Job: write a clear, friendly answer based *only* on retrieved content
- Output: draft answer in the user's language with source citations
- Why needed: turns raw retrieved chunks into a human-readable response

**Reviewer Agent**
- Input: draft answer + retrieved chunks
- Job: check the answer is accurate, grounded, and on-topic
- Output: quality score (1–10) + approved answer or revision request
- Why needed: catches hallucinations and off-topic responses before
  they reach the user

**Out-of-scope Handler**
- Triggered when: Reviewer scores below threshold OR question has no
  relevant chunks in the knowledge base
- Job: politely decline and suggest contacting the community directly
- Why needed: better to say "I don't know" than guess wrongly

### How agents communicate
Each agent receives the output of the previous agent as its input.
CrewAI handles this passing of context automatically.

---

## Data Flow — Full Example

User types: *"Wie viel kostet der Raum am Wochenende?"*
(German: "How much does the room cost on the weekend?")

1. **Language Detector** → detects German, passes question forward
2. **Retriever** → searches ChromaDB, finds pricing chunks
3. **Answer Agent** → writes answer in German from pricing chunks
4. **Reviewer** → scores 9/10, approves answer
5. User receives answer in German with source reference

---

## Folder Structure

    community-faq-bot/
    │
    ├── README.md                  ← project overview
    ├── DEVLOG.md                  ← development journal
    ├── requirements.txt           ← Python dependencies
    ├── .env.example               ← environment variables template
    │
    ├── docs/
    │   ├── architecture.md        ← this file
    │   └── decisions.md           ← technology choices and reasoning
    │
    ├── data/
    │   └── knowledge_base/        ← FAQ markdown files
    │       ├── pricing.md
    │       ├── room_specs.md
    │       ├── booking_rules.md
    │       └── cancellation.md
    │
    ├── src/
    │   ├── agents/                ← CrewAI agent definitions
    │   │   ├── __init__.py
    │   │   ├── language_detector.py
    │   │   ├── retriever.py
    │   │   ├── answer_agent.py
    │   │   └── reviewer.py
    │   │
    │   ├── rag/                   ← RAG pipeline
    │   │   ├── __init__.py
    │   │   ├── ingest.py          ← chunks + embeds documents
    │   │   └── retriever.py       ← searches ChromaDB
    │   │
    │   ├── ui/                    ← Streamlit interface
    │   │   └── app.py
    │   │
    │   └── __init__.py
    │
    └── tests/
        └── eval/                  ← evaluation scripts and test questions

---

## Key Design Decisions

For the reasoning behind each technology choice, see [decisions.md](decisions.md).

---

*This document is updated at the end of each build phase.*