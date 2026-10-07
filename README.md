# 🛠️ Cloud Log Troubleshooter

> SRMIST Capstone Project | Team: `Rajkumar C`-[RA2511028020072] 
## Problem Statement

Developers and cloud engineers spend significant time reading large log files to find the root cause of errors and failures, because the same error repeats thousands of times, real causes are buried among cascading symptoms, and the fix often depends on experience.

## Project Overview

Cloud Log Troubleshooter is a web app that takes raw application/cloud logs and automatically:

1. Parses them (timestamps, levels, stack traces)
2. Groups repeated errors into unique issues
3. Matches them against a built-in knowledge base of common cloud failures (timeouts, OOM, disk full, IAM, throttling, DNS, Kubernetes, etc.)
4. Highlights the **likely trigger** (earliest error) versus cascading failures
5. Suggests probable causes and fixes
6. *(Optional)* Uses any LLM (Claude, OpenAI, Gemini, Groq, OpenRouter, Ollama, ...) to explain the root-cause chain in plain language

The core analysis is rule-based and works offline. The AI explanation is an optional add-on and is provider-agnostic.

## Key Features

- Upload a log file, paste text, or use the built-in sample
- Supports common formats (`2026-10-07 10:02:41,882 ERROR ...`, ISO timestamps, `WARNING/FATAL` aliases)
- Stack-trace aware (continuation lines are attached to the right error)
- Smart grouping: `Timeout after 30000ms` and `Timeout after 5ms` become one issue
- Built-in knowledge base with 14 failure categories, each with cause + fix
- Root-cause heuristic and error-over-time chart
- Metrics dashboard: total lines, errors, warnings, error rate
- Optional "Explain with AI" checkbox that works with any provider
- Built-in debugging chat: ask follow-up questions, get plain-language explanations of stack traces, and concrete commands or fixes, grounded in your analysis and log excerpt
- Unit tests for parser and analyzer

## Technologies Used

| Area | Tech |
|---|---|
| Language | Python 3.10+ |
| Web UI | Flask |
| AI (optional) | Any LLM API: Anthropic, OpenAI, Gemini, or any OpenAI-compatible endpoint (stdlib HTTP, no SDK) |
| Testing | pytest |

## Project Structure

```
cloud-log-troubleshooter/
├── app.py                  # Flask web app
├── templates/index.html    # UI template
├── log_analyzer/
│   ├── parser.py           # log text -> LogEntry objects
│   ├── analyzer.py         # grouping, knowledge base, root cause
│   └── llm.py              # optional multi-provider AI explanation
├── sample_logs/app.log     # demo log
├── tests/test_analyzer.py
├── requirements.txt
└── .env.example
```

## Setup and Installation

```bash
git clone https://github.com/<your-username>/cloud-log-troubleshooter.git
cd cloud-log-troubleshooter

python -m venv .venv
# Windows: .venv\Scripts\activate
# Mac/Linux:
source .venv/bin/activate

pip install -r requirements.txt
```

**Optional (AI explanation):** no extra packages needed. Paste an API key in the app, or set environment variables: 

```bash
# Mac/Linux
export LLM_API_KEY=your_key_here
# Windows PowerShell
$env:LLM_API_KEY="your_key_here"
```

The provider is auto-detected from the key prefix (`sk-ant-` Anthropic, `sk-or-` OpenRouter, `AIza` Gemini, `gsk_` Groq, `sk-` OpenAI). Otherwise pick it from the dropdown or set `LLM_PROVIDER`. If the provider isn't detected automatically please choose the available name needed from the drop down menu.

| Provider | `LLM_PROVIDER` | Provider-specific key var |
|---|---|---|
| Anthropic (Claude) | `anthropic` | `ANTHROPIC_API_KEY` |
| OpenAI | `openai` | `OPENAI_API_KEY` |
| Google Gemini | `gemini` | `GEMINI_API_KEY` |
| Groq / OpenRouter / Mistral / DeepSeek / Together | `groq` / `openrouter` / `mistral` / `deepseek` / `together` | `GROQ_API_KEY`, etc. |
| Ollama / LM Studio (local, no key) | `ollama` / `lmstudio` | none |
| Any other OpenAI-compatible server | `custom` | `LLM_API_KEY` + `LLM_BASE_URL` + `LLM_MODEL` |

Override the model with `LLM_MODEL` (or the Model box in the UI).

## How to Run-after setting up the python enviorment.

```bash
python app.py
```

Open http://127.0.0.1:8501, keep **Sample log** selected and click **Analyze logs** for a demo, or upload / paste your own `.log` / `.txt` logs.

Run the tests:

```bash
python -m pytest
```

## How It Works

1. **Parse**: regex extracts timestamp, level, and message; indented lines are treated as stack traces.
2. **Normalize**: numbers, IPs, UUIDs, and hex values are replaced with placeholders to build a signature for each error.
3. **Group and classify**: identical signatures are counted and matched against the knowledge base (first match wins).
4. **Root cause**: the earliest error group is flagged as the likely trigger; later groups are often cascades.
5. **Explain and chat (optional)**: a compact summary of the grouped issues is sent to your chosen LLM for a plain-language root-cause chain. The chat assistant also receives up to ~12,000 characters of your log (start and end) so it can answer follow-up questions. Your API key stays in your browser tab (sessionStorage) and is only sent to this local app, which forwards it to the provider. Avoid logs containing secrets or personal data when using a hosted provider.

## screenshots are provided along
## Limitations and Future Work

- Knowledge base is rule-based; could be extended or learned from data
- Root-cause detection is heuristic (earliest error), not full causal analysis
- Add direct integration with AWS CloudWatch / Azure Monitor / GCP Logging
- Add anomaly detection and alerting

## License

For academic use (SRMIST Capstone).
