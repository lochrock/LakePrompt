
Final project video: https://www.youtube.com/watch?v=WTw-CfKQz2Y  
Final project report: https://docs.google.com/document/d/18_QpkRfaMsjdLDJJc2h_Xb8v9faGFAYT-95sEH-TGfY/edit?usp=sharing

# LakePrompt

LakePrompt is a Python research prototype for answering natural-language questions over a local directory of CSV files. It profiles columns, retrieves schema signals, plans joins, executes SQL over a local SQLite database, packages evidence for an LLM, and returns grounded answers with citations.

LakePrompt is now local-only. It does not download or prepare data from GitHub, ZIP URLs, or other remote sources. You must point it at a directory of CSV files already present on disk.

If you need example data to try locally, you can manually download the Spider Join dataset here:

- `https://github.com/superctj/spider-join-data/raw/main/data.zip`

## Quick Start

```bash
git clone git@github.com:jacob-erard-professional/LakePrompt.git
cd LakePrompt

python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt

export ANTHROPIC_API_KEY="your_api_key_here"
export HF_TOKEN="your_hf_token_here"

python3 demo.py --lake-dir /path/to/your/csv_directory
```

After initialization, `demo.py` drops into an interactive prompt. Type a question, or `q` to quit.

Example:

```text
> Ask a question about the CSV files in your local lake
> Ask another question
> q
```

## Current Scope

- local CSV-directory initialization only
- recursive CSV discovery from nested directories
- stable table naming for nested paths such as `region/sales/customers.csv -> region__sales__customers`
- semantic column retrieval with `sentence-transformers` and `hnswlib`
- join-path discovery with overlap and schema heuristics
- SQLite execution over the local lake
- TOON-packaged prompts for query planning and final answering
- local summary-cache persistence under `.lakeprompt_cache/`

## Repository Layout

- `lakeprompt/`: core package
- `demo.py`: the only supported interactive CLI
- `debug_one_question.py`: single-question debug runner with verbose tracing
- `data/`: local CSV schemas used by evaluation and debugging
- `eval/`: evaluation scripts and benchmark files
- `requirements.txt`: pinned dependencies
- `Documents/`: project documents

## Public API

```python
from lakeprompt import LakePrompt
```

Main entry points:

- `LakePrompt(...)`: initialize from a local CSV directory
- `LakePrompt.query(question)`: run the full retrieval/planning/execution pipeline

`LakePrompt.query(...)` returns a `LakeAnswer` with:

- `text`
- `evidence`
- `cited_ids`
- `raw_response`
- `prompt`

## Usage

### Python

```python
from lakeprompt import LakePrompt

lp = LakePrompt(
    lake_dir="/path/to/your/csv_directory",
    model="claude-sonnet-4-20250514",
    logger=True,
)

answer = lp.query("Ask a question about your local data")
print(answer.text)
print(answer.cited_ids)
print(answer.evidence)
```

### CLI

```bash
python3 demo.py --lake-dir /path/to/your/csv_directory
```

Useful flags:

- `--question "..."`: ask one question before entering interactive mode
- `--logger`: print pipeline traces to stdout
- `--show-prompt`: print the final packaged prompt
- `--show-sql`: print executed SQL for each evidence path
- `--cache-path ./summary_cache.json`: set an explicit summary-cache file
- `--cache-dir ./.lakeprompt_cache`: set the cache directory root
- `--no-save-artifacts`: disable summary-cache persistence for the run

## Debugging One Question

```bash
source .venv/bin/activate
export ANTHROPIC_API_KEY="your_api_key_here"
./.venv/bin/python debug_one_question.py
```

This is useful when you want to inspect:

- semantic retrieval results
- planner-visible schema context
- raw / coerced / sanitized query plans
- refined SQL
- final evidence rows

## Running Evaluation

```bash
source .venv/bin/activate
export ANTHROPIC_API_KEY="your_api_key_here"

python eval/evaluation.py \
  --dataset-root ./data \
  --questions-file ./eval/spider_join_questions_with_ground_truth.json \
  --output-json ./eval/results.json \
  --output-txt ./eval/results.md
```

Useful flags:

- `--max-questions 100`
- `--sample-rows 10`
- `--claude-model claude-sonnet-4-20250514`
- `--lakeprompt-model claude-sonnet-4-20250514`
- `--cache-path ./eval/summary_cache.json`
