# Ubuntu 26.04 Local & Free Automation Guide

This guide configures **Aider** and **Hermes** on **Ubuntu 26.04** using **Ollama** (for local, free processing) and **OpenRouter** (for free cloud models).

---

## 1. Environment & Prerequisites

Ensure your system is updated and standard Python tooling is ready.

```bash
sudo apt update && sudo apt install -y python3-pip python3-venv git pipx
pipx ensurepath
# Restart terminal or source your profile
source ~/.bashrc
```

---

## 2. Setting Up Ollama (100% Free Local Execution)

Ollama runs models locally on your Ubuntu machine. It is perfect for local tasks without rate limits.

### Step 1: Install Ollama
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Step 2: Pull Optimal Models
For coding with Aider or agent tasks with Hermes, pull a highly capable coding model like **Qwen2.5-Coder (7B or 14B)**:
```bash
ollama run qwen2.5-coder:7b
```
*(Type `/exit` once the model downloads to return to your shell. The background service remains active.)*

---

## 3. Configuring Aider (The Code builder)

Install Aider via `pipx` to keep its dependencies isolated.

```bash
pipx install aider-chat
```

### Workflow A: Pairing Aider with Local Ollama
Navigate to your coding directory, initialize Git, and launch Aider pointing to your local Ollama instance:
```bash
mkdir -p ~/projects/daily-automation && cd ~/projects/daily-automation
git init

# Launch Aider using Qwen2.5-Coder via Ollama
aider --model ollama/qwen2.5-coder:7b
```

### Workflow B: Pairing Aider with Free OpenRouter
If you want to leverage external models for free (like `meta-llama/llama-3.1-8b-instruct:free`), pass your OpenRouter API key:
```bash
export OPENROUTER_API_KEY="your_free_openrouter_key_here"
aider --model openrouter/meta-llama/llama-3.1-8b-instruct:free
```

---

## 4. Configuring Hermes (The Task Runner)

Hermes acts as your background scheduler. It relies on Python environments.

```bash
mkdir -p ~/hermes-runtime && cd ~/hermes-runtime
python3 -m venv venv
source venv/bin/activate
pip install hermes-agent # Or clone the repository depending on the distribution channel
```

### Connecting Hermes to OpenRouter / Ollama
Create an environment file (`.env`) inside your runtime folder:
```bash
# To use OpenRouter Free tier:
OPENROUTER_API_KEY="your_free_openrouter_key_here"
DEFAULT_MODEL="openrouter/meta-llama/llama-3.1-8b-instruct:free"

# OR to route Hermes tasks to local Ollama:
# DEFAULT_MODEL="ollama/qwen2.5-coder:7b"
```

---

## 5. The Workflow: From Concept to Automation

Here is how you use them together on Ubuntu:

### Step 1: Use Aider to Build the Automation Script
Open Aider in your project folder and ask it to write an automation script (e.g., a script to clean up your `~/Downloads` folder every day or scrape a webpage):
```bash
aider --model ollama/qwen2.5-coder:7b
```
*Prompt to Aider:* 
> "Write a python script called `cleanup.py` that moves files older than 7 days from ~/Downloads to an archive folder. Add error logging."

Aider will write the code and automatically commit it to Git.

### Step 2: Register & Schedule with Hermes
Move or reference that script in your Hermes-monitored task suite. You can use Ubuntu's system native `systemd` or a user cron expression to keep Hermes evaluating or running your new automated "skill" continuously in the background.

```bash
# To test your script inside the automation suite:
python3 ~/projects/daily-automation/cleanup.py
```
