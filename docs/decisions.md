# Decision Log

This document captures the key technology and architecture choices made
during the project, and the reasoning behind each one.

The goal is to explain *why* we built it this way — not just what we built.

---

## Decision 1 — Python as the primary language

**Chosen:** Python 3.11.9
**Alternatives considered:** JavaScript/Node.js

**Reasoning:**
Python is the dominant language in data engineering and AI/ML.
Every major AI framework (CrewAI, LangChain, ChromaDB, Streamlit)
has Python as its first-class citizen. Node.js support is secondary
or non-existent for most of these tools.

For a project that also serves as a portfolio piece for a data
engineering career transition, Python is the only sensible choice.

**Version choice — why 3.11.9 specifically:**
3.11 is the most stable version for the dependency stack we're using.
3.12+ has some compatibility issues with certain AI libraries.
3.10 is older than needed. 3.11.9 is the sweet spot.

---

## Decision 2 — pyenv for Python version management

**Chosen:** pyenv
**Alternatives considered:** Direct python.org install, Conda, asdf

**Reasoning:**
Direct installation from python.org puts Python in a fixed system location.
This causes problems when projects need different Python versions, and
risks interfering with macOS's own Python installation.

pyenv installs Python versions in isolated locations and lets you switch
between them per project. It's the professional standard on macOS for
Python development.

Conda was considered but is heavier than needed — it's designed for
data science environments with complex dependencies. We don't need that.

asdf is a more general version manager that handles multiple languages,
but pyenv is simpler for a Python-focused project.

---

## Decision 3 — CrewAI as the agent framework

**Chosen:** CrewAI
**Alternatives considered:** LangGraph, AutoGen, plain LangChain

**Reasoning:**

| Framework | Verdict |
|---|---|
| **LangGraph** | More flexible but steeper learning curve. Better for complex non-linear workflows. Overkill for a focused FAQ bot. |
| **AutoGen** | Effectively in maintenance mode — Microsoft shifted focus elsewhere. Not a safe long-term choice. |
| **Plain LangChain** | Too low-level. Would require building agent orchestration from scratch. |
| **CrewAI** | Best balance of simplicity and capability for a first agentic project. Role-based agents map naturally to our use case. |

CrewAI's model: define agents with roles, give them tasks, let the
framework handle coordination. This matches how we think about the
problem — a language detector, a retriever, an answer writer, a reviewer.

**Future consideration:**
LangGraph would be worth learning as a second project. It handles
non-linear workflows better and is used more in production systems.

---

## Decision 4 — MiniMax M2.5 as the AI model

**Chosen:** MiniMax M2.5 via OpenRouter (free tier)
**Alternatives considered:** Claude Sonnet, GPT-4o, Gemini Flash, Llama local

**Reasoning:**
This is a learning project and POC. The primary constraints are:
- Zero cost during development
- Good enough quality for FAQ answering
- Easy integration with CrewAI

MiniMax M2.5 via OpenRouter hits all three:
- Free tier available (rate limited but fine for development)
- 196K context window — more than enough for RAG chunks
- OpenAI-compatible API — works with any framework that supports OpenAI

**Why not Claude/GPT-4o for development:**
Claude Sonnet and GPT-4o are higher quality but cost money per call.
For a learning project where you make hundreds of test calls while
debugging, cost adds up quickly. Better to learn on free, then
upgrade to paid if/when deploying publicly.

**Why not local models (Llama, Gemma):**
The development machine is a 2018 MacBook Pro with 16GB RAM.
Running local models on this hardware would be too slow for
comfortable development. Cloud API is the right choice here.

**Production consideration:**
If deployed publicly, would evaluate Claude Haiku or Sonnet for
better quality and EU data residency options.

**GDPR note:**
MiniMax is a Chinese company — data sent to their API leaves the EU.
Acceptable for a POC and internal testing. Would need proper data
processing agreement and privacy policy disclosure before public launch.

---

## Decision 5 — OpenRouter as the API gateway

**Chosen:** OpenRouter
**Alternatives considered:** Direct MiniMax API, direct Anthropic API

**Reasoning:**
OpenRouter provides a single API key and endpoint for 300+ AI models.
This means switching models in the future is one line of code change —
just change the model name string.

**Cost:**
OpenRouter passes through provider pricing without markup on inference.
Only cost is a 5.5% fee on credit purchases — negligible at our scale.
MiniMax M2.5 free tier is available directly through OpenRouter.

**Future flexibility:**
As the project grows, we may want to:
- Compare MiniMax vs Claude output quality side by side
- Upgrade to a paid model for production
- Use different models for different agents

OpenRouter makes all of this trivial. Direct API integration with
one provider would require code changes for each switch.

---

## Decision 6 — ChromaDB as the vector database

**Chosen:** ChromaDB
**Alternatives considered:** Pinecone, Weaviate, pgvector, Qdrant, FAISS

**Reasoning:**

| Option | Why not chosen |
|---|---|
| **Pinecone** | Cloud-only, costs money, overkill for a POC |
| **Weaviate** | More complex setup, better for larger scale |
| **pgvector** | Requires PostgreSQL setup — extra complexity |
| **Qdrant** | Good option but more complex than ChromaDB for beginners |
| **FAISS** | No persistence — data lost when program stops |
| **ChromaDB** | Runs locally, persists to disk, simple Python API, free |

ChromaDB runs entirely on your machine with no external service.
For a local development project this is ideal — no accounts, no
costs, no network dependency.

**Future consideration:**
For a production deployment with high query volume, Qdrant or
pgvector would be worth evaluating. ChromaDB is excellent for
development and small-scale production.

---

## Decision 7 — Streamlit for the UI

**Chosen:** Streamlit
**Alternatives considered:** Flask, FastAPI + React, Gradio

**Reasoning:**
Streamlit lets you build interactive web UIs in pure Python.
No HTML, no CSS, no JavaScript required.

For a project focused on learning AI/data engineering concepts,
spending time on frontend development would be a distraction.
Streamlit lets us get a working chat interface in under 50 lines
of Python.

**Limitations acknowledged:**
Streamlit is not suitable for high-traffic production apps — it
has session management limitations and isn't designed for
concurrent users at scale. For a community FAQ bot with modest
traffic, it's perfectly adequate.

**Future consideration:**
If the bot grows and needs a more polished UI, the backend
(CrewAI + RAG pipeline) can stay exactly as built. Only the
UI layer needs replacing — a clean separation of concerns.

---

## Decision 8 — Sequential agents over parallel

**Chosen:** Sequential agent execution (default CrewAI)
**Alternatives considered:** Parallel agent execution

**Reasoning:**
In our workflow, each agent depends on the previous agent's output:
- Retriever needs the language detector's output
- Answer agent needs the retriever's chunks
- Reviewer needs the answer agent's draft

There is no step where two agents can usefully work simultaneously.
Parallel execution adds complexity with no benefit here.

**RAM consideration:**
The development machine has 16GB RAM. Sequential execution means
only one model is active at a time — much more memory efficient.

---

## Decision 9 — Public GitHub repo

**Chosen:** Public repository
**Alternatives considered:** Private repository

**Reasoning:**
This project is explicitly a portfolio piece for a career transition
from data analyst to data engineer with AI skills.

A private repo is invisible to hiring managers, recruiters, and
the broader developer community. The entire learning value of
building this publicly — LinkedIn posts, GitHub profile signal,
potential community contributions — disappears with a private repo.

The project contains no sensitive data. API keys are in `.env`
(gitignored). FAQ content is fictional for now. There is no
reason to keep it private.

---

## Decisions Still Open

These decisions are deferred until later phases:

| Decision | When | Options |
|---|---|---|
| Embedding model choice | Phase 2 | OpenAI vs nomic-embed-text vs multilingual-e5 |
| Chunking strategy | Phase 2 | Fixed size vs semantic vs recursive |
| Reviewer scoring threshold | Phase 4 | What score triggers out-of-scope handler? |
| Production hosting | Phase 6+ | Hetzner VPS vs Mac Mini home server vs cloud |
| Production model | Phase 6+ | MiniMax paid vs Claude Haiku vs Gemini Flash |

---

*Updated as new decisions are made throughout the project.*