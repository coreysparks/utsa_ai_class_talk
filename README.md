# Small Models, Large Data, and the Unglamorous Work in Between

Slides and notes for a guest talk on AI and machine learning in applied settings, given in a colleague's graduate AI course at UT San Antonio.

Corey Sparks, Department of Sociology and Demography, UT San Antonio

## What the talk covers

The talk draws on applied work at the Coleridge Initiative, where state agency staff analyze linked administrative records (workforce, education, corrections, public benefits) inside a secure environment. It separates three uses of AI that tend to get lumped together:

1. **Small local models** for audience and market analysis: turning resumes and application statements into structured features with sentence embeddings, zero-shot classification, and small generative models run through Ollama on a laptop.
2. **AI as a collaborator in data engineering**: using ChatGPT and Claude Code to design and debug SQL and R pipelines against large administrative schemas, illustrated with a wage window bug and a simulation of how much it distorts an estimate.
3. **AI-assisted synthetic data**: a corrections dataset that ChatGPT generated from three plain-language prompts and that was used in two analyst training courses. The model met every marginal distribution it was asked for but dropped structural and conditional requirements without saying so.

The common thread is that each use needs a different kind of checking, and that output that runs, parses, or matches its marginals is not the same as output that is correct.

## Files

| Path | What it is |
|---|---|
| `files/ai_applied_talk.qmd` | Quarto source for the slides (reveal.js) |
| `files/ai_applied_talk.html` | Rendered slides; self-contained, opens in any browser |
| `files/utsa-slides.scss` | UTSA slide theme |
| `local_models_applied_talk.md` | Long-form notes that the slides were developed from |
| `synthetic data bg.md` | Background on the synthetic data projects |

## Viewing the slides

Open `files/ai_applied_talk.html` in a browser. Press `s` for speaker notes and `Esc` for a slide overview.

Small orange-bordered boxes marked CONFIRM are notes to the presenter about details still being verified. Remove them before the final version.

## Rendering

The slides need [Quarto](https://quarto.org) and a Python environment with Jupyter, because a few slides draw their figures with matplotlib at render time.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install jupyter numpy pandas matplotlib
quarto render files/ai_applied_talk.qmd
```

The appendix code (embeddings, zero-shot classification, and Ollama examples) is shown but not run, so its packages are not needed to render.

## Data

No data are included in this repository. The figures are either simulated in the slide code or computed from project files and written into the slides as numbers, with a comment noting the source. The resume and application-statement corpora contain personal information and are not shared. The synthetic corrections and Medicaid datasets live in separate course repositories.
