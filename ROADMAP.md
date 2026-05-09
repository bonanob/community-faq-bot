# Roadmap — Community FAQ Bot

The complete build plan from start to finish.
Updated as phases complete.

---

## Phase 1 — Foundation ✅
*Setting up everything needed before writing any code.*

### Environment setup
- [x] Install Homebrew
- [x] Install pyenv
- [x] Install Python 3.11.9 (with xz fix)
- [x] Install VS Code + extensions (Python, Pylance, GitLens)
- [x] Install `code` shell command
- [x] Configure Git with name and email
- [x] Generate SSH key pair (ed25519)
- [x] Add SSH public key to GitHub
- [x] Verify SSH authentication

### Repository setup
- [x] Create public GitHub repo `community-faq-bot`
- [x] Initialize with README, Python .gitignore, MIT license
- [x] Clone repo locally to ~/Documents/projects/

### Documentation
- [x] Write README.md (project overview, status, tech stack)
- [x] Write DEVLOG.md (session 1 setup notes)
- [x] Write docs/architecture.md (system design)
- [x] Write docs/decisions.md (technology choices)
- [x] Commit and push all documentation

### Project workflow
- [x] Migrate to Claude Projects for persistent context
- [x] Upload documentation files as project knowledge
- [x] Establish documentation workflow (DEVLOG raw, README clean)

---

## Phase 2 — Knowledge Base
*Building the RAG pipeline foundation.*

### Project structure
- [ ] Create folder structure (src/agents, src/rag, src/ui, data/knowledge_base, tests)
- [ ] Create __init__.py files in src folders
- [ ] Create .env.example with placeholder values
- [ ] Verify .env is in .gitignore

### Python environment
- [ ] Create virtual environment (python -m venv venv)
- [ ] Activate venv and verify isolation
- [ ] Install dependencies (crewai, chromadb, streamlit, openai, python-dotenv)
- [ ] Generate requirements.txt (pip freeze)
- [ ] Commit requirements.txt

### OpenRouter setup
- [ ] Create OpenRouter account
- [ ] Generate API key
- [ ] Add key to .env file (never commit)
- [ ] Test first MiniMax API call (test_connection.py)
- [ ] Verify bilingual response (English + German)

### Knowledge base content
- [ ] Decide on FAQ topics (pricing, room specs, booking rules, cancellation)
- [ ] Write data/knowledge_base/pricing.md
- [ ] Write data/knowledge_base/room_specs.md
- [ ] Write data/knowledge_base/booking_rules.md
- [ ] Write data/knowledge_base/cancellation.md
- [ ] Document FAQ structure conventions in DEVLOG

### RAG ingestion pipeline
- [ ] Choose embedding model (research OpenAI vs nomic vs multilingual-e5)
- [ ] Implement document loader (read markdown files)
- [ ] Implement chunking strategy (size + overlap)
- [ ] Implement embedding generation
- [ ] Implement ChromaDB storage and persistence
- [ ] Build src/rag/ingest.py
- [ ] Test ingestion with sample queries
- [ ] Update docs/decisions.md with embedding/chunking choices

---

## Phase 3 — First Agent
*One AI agent answering questions with RAG. No CrewAI yet.*

### Single-agent baseline
- [ ] Implement retrieval function (src/rag/retriever.py)
- [ ] Build basic prompt template with retrieved context
- [ ] Make end-to-end API call: question → retrieve → answer
- [ ] Test with English questions
- [ ] Test with German questions
- [ ] Verify citations and groundedness

### Quality validation
- [ ] Create test set of 20 sample questions (10 EN, 10 DE)
- [ ] Manually evaluate baseline responses
- [ ] Document baseline quality in DEVLOG
- [ ] Identify common failure patterns

### Lessons learned checkpoint
- [ ] Update DEVLOG with Phase 3 learnings
- [ ] Update README status table
- [ ] Write LinkedIn post about RAG basics
- [ ] Commit and push

---

## Phase 4 — Multi-Agent System
*Full CrewAI crew with specialized agents.*

### Agent definitions
- [ ] Implement Language Detector agent
- [ ] Implement Retriever agent (wraps RAG)
- [ ] Implement Answer Agent
- [ ] Implement Reviewer agent with scoring
- [ ] Implement Out-of-scope Handler

### Crew orchestration
- [ ] Define CrewAI tasks for each agent
- [ ] Configure sequential task execution
- [ ] Implement context passing between agents
- [ ] Build src/agents/crew.py main entry point

### Reviewer scoring system
- [ ] Define scoring criteria (groundedness, relevance, completeness)
- [ ] Define structured output format (score + reason + escalate flag)
- [ ] Set escalation threshold
- [ ] Build fallback to out-of-scope handler

### Quality validation
- [ ] Run test set through multi-agent system
- [ ] Compare quality vs single-agent baseline
- [ ] Document improvements and regressions
- [ ] Update docs/decisions.md with agent design rationale

---

## Phase 5 — Streamlit UI
*Simple bilingual chat interface.*

### Basic interface
- [ ] Set up Streamlit app structure (src/ui/app.py)
- [ ] Build chat input + message history
- [ ] Connect UI to crew pipeline
- [ ] Display answers with source citations
- [ ] Show language detection result

### Polish
- [ ] Add loading states (agents thinking...)
- [ ] Handle errors gracefully
- [ ] Add reset/clear chat button
- [ ] Style with basic theming
- [ ] Add usage notes / disclaimer

### Local demo
- [ ] Run Streamlit locally
- [ ] Test with realistic user questions
- [ ] Screenshot for README
- [ ] Record short demo video

---

## Phase 6 — Evaluation Framework
*Systematic quality measurement.*

### Test infrastructure
- [ ] Build expanded test set (50+ questions across topics)
- [ ] Define quality metrics (groundedness, accuracy, language match)
- [ ] Implement automated evaluation script
- [ ] Build evaluation report generator

### Observability
- [ ] Add logging across the pipeline (queries, retrievals, scores, latency)
- [ ] Track API token usage and costs
- [ ] Build basic monitoring dashboard
- [ ] Document evaluation results

### Documentation finalization
- [ ] Update README with results and benchmarks
- [ ] Add architecture diagram (visual, not ASCII)
- [ ] Write final blog-style post on LinkedIn
- [ ] Tag v1.0 release on GitHub

---

## Phase 7 — Validation with Community
*Show real users, gather feedback.*

- [ ] Demo to community members
- [ ] Collect feedback on accuracy and usefulness
- [ ] Document what works and what doesn't
- [ ] Decide on Phase 8 (deploy publicly?) based on feedback

---

## Phase 8 — Production Deployment (Optional)
*Only if community validates and wants to deploy.*

This phase is deferred and unscoped until Phase 7 completes.
Will involve: legal review (GDPR, AI Act), hosting decision 
(home server vs cloud VPS), production model upgrade, 
proper monitoring, payment integration, public website embed.

---

