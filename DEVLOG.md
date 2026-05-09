# Dev Log — Community FAQ Bot

A running journal of what I did, what broke, and what I learned.
Written in plain language for my own reference.

---

## Session 1 — Project Planning & Environment Setup
**Date:** May 2026

### What I decided to build
A bilingual (English + German) FAQ chatbot for a community room booking system.
Users can ask questions like "How much does the room cost on weekends?" and get
accurate answers based on a knowledge base I write myself.

The bot will be built using a technique called **RAG** (Retrieval-Augmented Generation)
combined with **AI agents**. I chose this project because it's a real problem,
teaches valuable skills for my data engineering career transition, and is
small enough to actually finish.

### The tech stack I chose and why

| Tool | What it does | Why I chose it |
|---|---|---|
| **CrewAI** | Orchestrates multiple AI agents | Beginner-friendly, good documentation |
| **MiniMax M2.5** | The AI model that answers questions | Free tier via OpenRouter — zero cost for learning |
| **OpenRouter** | Gateway to access MiniMax and other AI models | One API key for all models, easy to switch |
| **ChromaDB** | Stores and searches the knowledge base | Free, runs locally, simple to use |
| **Streamlit** | The chat interface users see | Pure Python, fast to build with |
| **Python 3.11.9** | The programming language | Most stable version for our dependencies |
| **pyenv** | Manages multiple Python versions | Professional standard on macOS |

### What RAG actually is
RAG means giving the AI access to *my specific documents* at the time it answers a question.
Without RAG, the AI only knows what it was trained on — it has no idea about our
community room's pricing, rules, or availability.

With RAG:
1. I write FAQ documents (pricing, rules, room specs etc.)
2. Those documents get broken into chunks and stored in ChromaDB as vectors
3. When a user asks a question, the system finds the most relevant chunks
4. Those chunks get sent to the AI along with the question
5. The AI answers using *my actual information* — not guesses

This prevents hallucination (the AI making things up) because it can only answer
from retrieved content.

### What AI agents are
Agents are AI models with specific roles that work together on a task.
Instead of one AI doing everything, each agent has one clear job:

- **Language Detector** — figures out if the user wrote in English or German
- **Retriever** — searches ChromaDB for relevant FAQ content
- **Answer Agent** — writes the response based on retrieved content
- **Reviewer** — checks the answer is accurate and not made up
- **Out-of-scope Handler** — politely declines questions the bot can't answer

CrewAI is the framework that coordinates these agents and passes information between them.

---

### Environment setup — what I actually did

#### Step 1 — Homebrew
Homebrew is macOS's package manager. Think of it like an App Store for developer tools.
Everything else installs through it.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After install, ran the two "Next steps" commands it showed to add Homebrew to PATH.
PATH is the list of places the terminal looks for programs when you type a command.

#### Step 2 — pyenv + Python
pyenv lets you install and switch between multiple Python versions without breaking things.
Professionals use this instead of installing Python directly.

First installed a required library *before* Python (learned this the hard way):
```bash
brew install xz
```

**Error I hit:** Got `ModuleNotFoundError: No module named '_lzma'` when first trying
to install Python. This happened because `xz` wasn't installed yet when Python compiled.
Fix: install `xz` first, then uninstall and reinstall Python.

Then installed pyenv and configured the shell:
```bash
brew install pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

The `~/.zshrc` file is the terminal's config file. It runs every time you open Terminal.
These lines tell the shell where to find pyenv. Had to fully close and reopen Terminal
for changes to take effect.

Then installed Python:
```bash
pyenv install 3.11.9
pyenv global 3.11.9
```

`pyenv global` sets the default Python version for the whole machine.

#### Step 3 — VS Code
Downloaded VS Code from code.visualstudio.com.

Extensions installed:
- **Python** (by Microsoft) — auto-installed Pylance too
- **GitLens** (by GitKraken) — shows who changed what line by line. Has a Pro tier
  but the free version is all I need. Just clicked "Continue with Free".

Installed the `code` shell command via Cmd+Shift+P → "Shell Command: Install 'code' command in PATH".
This lets me type `code .` in Terminal to open VS Code in the current folder.
The `.` in bash always means "current directory".

#### Step 4 — Git + SSH
Configured Git with my identity (stamps every commit with name + email):
```bash
git config --global user.name "My Name"
git config --global user.email "my@email.com"
```

Generated an SSH key pair — this lets my Mac authenticate to GitHub without passwords:
```bash
ssh-keygen -t ed25519 -C "my@email.com"
```

SSH works like a lock and key:
- **Private key** — stays on my Mac, never shared
- **Public key** — given to GitHub (the "lock")
- When connecting, GitHub checks if my private key matches — if yes, I'm in

Copied the public key cleanly using `pbcopy` (macOS clipboard tool):
```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

**Error I hit:** GitHub said "Key is invalid" when I pasted. Cause was an incomplete
copy — hidden characters or line breaks got included. Fix: use `pbcopy` instead of
manually selecting text.

Added the public key to GitHub → Settings → SSH and GPG Keys → New SSH Key.
Named it `mbp2018` to know which machine it belongs to.

Verified it worked:
```bash
ssh -T git@github.com
# Response: Hi bonanob! You've successfully authenticated...
```

#### Step 5 — GitHub repo + local clone
Created repo on github.com:
- Name: `community-faq-bot`
- Public (portfolio piece)
- Initialized with README, Python .gitignore, MIT license

Cloned locally using SSH:
```bash
cd ~/Documents
mkdir projects
cd projects
git clone git@github.com:bonanob/community-faq-bot.git
cd community-faq-bot
code .
```

`git clone` downloads the repo to my machine and sets up the Git connection.
`code .` opens VS Code in the current folder.

---

### Things I learned today

- **Nobody memorizes bash commands.** Developers have muscle memory from repetition,
  personal setup docs, and Google. The goal is understanding *why*, not memorizing *what*.

- **The `.` means current directory** in bash. Shows up everywhere: `code .`, `git add .`, `ls .`

- **pyenv vs direct Python install** — pyenv is better because it isolates versions per project
  and doesn't touch the system Python macOS depends on.

- **SSH key pairs** — public key is the lock (give it out), private key is the key (never share).
  `pbcopy` is the safe way to copy keys on macOS.

- **README-driven development** — write what you're building before writing code.
  Forces clarity of thought.

---

### What's next
- Create the project folder structure
- Set up Python virtual environment
- Install dependencies (CrewAI, ChromaDB, Streamlit, OpenAI SDK)
- Set up OpenRouter account and API key
- Test first MiniMax API call
- Write initial README.md
- Write the first FAQ knowledge base documents

---