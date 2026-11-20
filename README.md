# Generative AI Programming Practicals

Weekly lab work for the Generative AI course (B.Tech CSE, KLH Bachupally). Each week builds on the last: first API call, provider abstraction, usage logging, tokenizers, cost budgeting, and a first evaluation baseline.

## Repository Structure

```
gen-ai/
├── README.md
├── .gitignore
└── Practical/
    ├── week1api/
    ├── week2-genai/
    ├── week3/
    │   └── Week3_Gemini_MongoDB_Logging.ipynb
    ├── week4-tokenizer/
    ├── TokenEconomics/        (Week 5)
    └── week6/
```

## Weeks at a Glance

| Week | Folder | Topic | Stack |
|------|--------|-------|-------|
| 1 | `week1api` | First LLM API call | Gemini API, python-dotenv |
| 2 | `week2-genai` | Provider abstraction (cloud and local) | Gemini, Ollama, unittest |
| 3 | `week3` | API usage logging, cost and latency analysis | Colab, Gemini, MongoDB Atlas, pandas, matplotlib |
| 4 | `week4-tokenizer` | BPE vs WordPiece vs SentencePiece | Hugging Face Transformers |
| 5 | `TokenEconomics` | Cost and rate-limit budgeting | Python, python-dotenv, unittest |
| 6 | `week6` | Golden question set and first baseline, Release v0.1 | Ollama (Llama 3.2), requests |

## Setup (applies to every week)

Requires Python 3.10 or later. Each week has its own `requirements.txt`, so create a virtual environment inside the week folder you are working on.

```powershell
cd Practical\<week-folder>
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

If PowerShell blocks activation, run this once:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

Secrets and local settings live in a `.env` file, which is excluded from version control by `.gitignore`. Never commit API keys.

---

## Week 1: First LLM API Call

**Folder:** `Practical/week1api`

A minimal script that loads a Gemini API key from `.env`, sends one prompt, and prints the response.

**Setup:** create `.env` in the folder:

```
GEMINI_API_KEY=your_key_here
```

**Run:**

```powershell
python main.py
```

## Week 2: Provider Abstraction

**Folder:** `Practical/week2-genai`

A small client layer that puts different LLM backends behind one interface.

- `llm_client.py` defines the abstract `LLMProvider`, with `GeminiProvider` (cloud) and `OllamaProvider` (local Llama 3.2), plus `LLMClient`, which works with any provider.
- `main.py` asks which provider to use, takes a question, and prints the answer.
- `test_client.py` tests `LLMClient` with a fake provider, so no API call is needed.

**Setup:** `GEMINI_API_KEY` in `.env` for Gemini. For Ollama, run `ollama pull llama3.2` and keep Ollama running.

**Run:**

```powershell
python main.py
python -m unittest
```

## Week 3: Usage Logging with MongoDB

**Folder:** `Practical/week3`
**File:** `Week3_Gemini_MongoDB_Logging.ipynb` (runs in Google Colab)

A notebook that wraps Gemini calls with production-style monitoring.

- Calls Gemini and records input, output, and total tokens for every attempt.
- Estimates cost per request from per-million-token prices.
- Retries temporary errors (408, 429, 5xx, timeouts) with exponential backoff and jitter.
- Writes each attempt to MongoDB Atlas as a log record, using an idempotent upsert so retries do not create duplicates.
- Analyses the logs with an aggregation pipeline, a pandas DataFrame, and latency and token-usage charts.

**Setup:** add `GEMINI_API_KEY` and `MONGODB_URI` as Colab Secrets (the key icon in the left sidebar) and enable notebook access. Then run the cells top to bottom.

## Week 4: Tokenizer Comparison

**Folder:** `Practical/week4-tokenizer`

Compares three real pretrained tokenizers on English, Telugu, and Hindi text, printing the tokens, token IDs, and token counts.

| Tokenizer | Model |
|-----------|-------|
| BPE | `gpt2` |
| WordPiece | `bert-base-multilingual-cased` |
| SentencePiece | `xlm-roberta-base` |

The differences in token counts come from each model's vocabulary and training data as well as the algorithm. GPT-2 is mainly English-trained, so Telugu and Hindi split into many more tokens.

**Run** (internet needed on the first run to download tokenizer files):

```powershell
python tokenizer_comparison.py
```

## Week 5: Token Economics

**Folder:** `Practical/TokenEconomics`

A budget planner to run before building an LLM feature. Given traffic and token assumptions, it estimates daily and monthly cost, checks peak RPM and TPM against provider limits with a safety margin, and prints a go/no-go recommendation.

**Setup:** create `.env` with the planning values:

```
INPUT_PRICE_PER_MILLION=0.30
OUTPUT_PRICE_PER_MILLION=1.20
REQUESTS_PER_DAY=2000
AVG_INPUT_TOKENS=1200
AVG_OUTPUT_TOKENS=300
PEAK_REQUESTS_PER_MINUTE=30
RPM_LIMIT=60
TPM_LIMIT=100000
MONTHLY_BUDGET_USD=50
DAILY_BUDGET_USD=2
SAFETY_MARGIN_PERCENT=20
```

These are illustrative values, not real provider rates. Replace them with the current rate card for your model.

**Run:**

```powershell
python budget.py
python -m unittest
```

**Result with the sample values:** about 3,000,000 tokens/day, $1.44/day, $43.20/month. With the 20% margin the monthly cost is $51.84, which is over the $50 budget, so the monthly check reports REVIEW.

## Week 6: Golden Set Baseline (Release v0.1)

**Folder:** `Practical/week6`

The first evaluation baseline for a local model.

- `golden_set.json` holds 30 questions on tokenization, generation, embeddings, and Transformers, each with an expected answer, scoring keywords, and a source link.
- `main.py` sends every question to Llama 3.2 through the Ollama API at temperature 0, scores each answer by keyword coverage (PASS at 60% or more, otherwise REVIEW), and saves everything to `baseline_results.csv`.

**Setup:** install [Ollama](https://ollama.com) and pull the model:

```powershell
ollama pull llama3.2
```

**Run:**

```powershell
python main.py
```

The summary reports passed, review, and failed counts and average keyword coverage. This run is labelled **Release v0.1**.

**Notes:**

- On low-RAM machines (8 GB), close other apps first. Use `llama3.2:1b` if the 3B model is too slow, and record which model you used.
- Keyword matching is a coarse metric. A correct answer worded differently from the keywords can still score REVIEW, so read the results alongside the expected answers.

## Author

Santhosh, KLH Bachupally
