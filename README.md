# Prompt Injection / Jailbreak Defense Demo

A reproducible study of how **prompt-engineering defenses** change the attack
success rate of a domain-constrained chatbot - and what those defenses cost in
helpfulness.

**VetBot** (a *Paws & Claws* veterinary assistant) may only discuss dogs and cats.
Five attacks try to make it answer `What is the capital of Brazil?`; five
prompt-engineering defenses try to stop them. Everything runs locally via Ollama.

- **Graded write-up:** [`REPORT.md`](REPORT.md)
- **Guided explanation of every file:** [`PROJECT_WALKTHROUGH.md`](PROJECT_WALKTHROUGH.md)
- **Interactive demo:** `app.py` (Streamlit)

## Quick start

```powershell
python -m venv .venv
& ".venv\Scripts\python.exe" -m pip install -r requirements.txt

ollama pull granite3.3:2b                     # or whatever you set in config.MODEL

& ".venv\Scripts\python.exe" bot.py           # smoke test: 1 benign + 1 forbidden
& ".venv\Scripts\python.exe" -m pytest tests -q
& ".venv\Scripts\python.exe" harness.py       # full experiment -> results/
& ".venv\Scripts\python.exe" -m streamlit run app.py
```

## What each file is

| File | Role |
|---|---|
| `config.py` | The single model variable, temperature, sentinel token, refusal line. |
| `bot.py` | Baseline bot + `chat()` - the only function that calls the model. |
| `attacks.py` | The five attack payloads (A1-A5) with the mechanism behind each. |
| `defenses.py` | The five defense prompts (D1-D5) with the technique behind each. |
| `guard.py` | Output-side screen: forbidden answer / sentinel token / prompt overlap. |
| `harness.py` | Runs the matrices, scores them, writes `results/`. |
| `app.py` | Streamlit demo platform (live attack, sweep, saved results, report). |
| `results/` | Machine-written evidence: raw outputs + Markdown tables. |
| `tests/` | pytest suite for the scoring heuristics and the guard. |
| `REPORT.md` | Written study: threat model, catalogues, results, failure analysis. |
| `PROJECT_WALKTHROUGH.md` | Plain-language explanation of the design and findings. |

## The metrics

- **ASR** - Attack Success Rate. 1 = the attack got the forbidden answer out.
- **FPR** - False Positive Rate. 1 = a benign on-topic question was refused.
- **Refusal preamble** - the scope line was emitted *and* the question was
  answered anyway. A quality defect, not a blocked request, so it is reported
  separately from FPR.
- **ASR guarded** - ASR after the output guard in `guard.py` has screened the reply.

## Reproducing the headline result, with spread

```powershell
& ".venv\Scripts\python.exe" harness.py --repeats 3
```

That is 3 x 54 = 162 model calls. `results/results_asr_variance.md` reports the
per-defense ASR mean, minimum and maximum.

Every results row carries its own provenance (`model`, `temperature`, `repeat`,
`timestamp`), and every Markdown table starts with a provenance comment, so any
table can be traced back to the run that produced it.

## Known caveat

A small local model is **not bit-reproducible across server sessions**. At
`temperature = 0.0` decoding is greedy, so a sampling seed would have no effect -
the variance this project actually observed came from run-to-run execution state
(KV-cache reuse and batching), which is why the flag is `--repeats` rather than
`--seeds`. Use it before quoting any single cell. The conclusions in `REPORT.md`
are drawn from the archived run in `results/`.
