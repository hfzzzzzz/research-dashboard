# {{Project name}}

> {{One sentence: what this project investigates and the target metric. e.g. "Predicting molecular dipole moments on QM9 with GNNs; target test MAE < 0.30 D."}}

<!--
  How to use this file (delete this comment afterwards):
  1. Copy to the root of the research repository as CLAUDE.md
  2. Replace every {{placeholder}}; delete sections that do not apply, but KEEP
     "Logging conventions" and "research/ data files"
  3. Copy the starter research/*.json files from this platform's templates/ directory
  4. This file has two jobs: it is the working brief for Claude Code AND the source
     rendered on the dashboard's Methods tab
-->

## Project overview

- **Field**: {{area}}
- **Background**: {{why this problem, what existing work misses}}
- **Current stage**: {{e.g. baseline reproduction / method improvement / writing}}
- **Timeframe**: {{YYYY-MM to YYYY-MM}}

## Research questions and hypotheses

- **RQ1**: {{question}}
  - **H1**: {{testable hypothesis with an expected effect size, e.g. "method X improves metric Y by at least 8%"}}
- **RQ2**: {{question}}

## Method

### Overall approach

{{How the method works, how variables are controlled, what makes the conclusion trustworthy}}

### Experimental design

1. {{Data splits and seeds, e.g. train/val/test = 8/1/1, seed 42}}
2. {{Repetitions and reporting, e.g. 3 seeds per configuration, mean ± sd}}
3. {{Stopping criteria, hyper-parameter ranges}}

### Data

- **Source**: {{dataset name, version, how to obtain it}}
- **Size**: {{samples, dimensions}}
- **Preprocessing**: {{script path and key steps}}

### Analysis and evaluation

- **Primary metric**: {{e.g. test MAE}}; **secondary**: {{e.g. R², bucketed error}}
- **Significance**: {{e.g. paired t-test, p < 0.05}}
- **Baselines**: {{what this is compared against}}

## Environment and reproduction

```bash
{{setup and a representative run, e.g.
conda env create -f environment.yml
python train.py --config configs/base.yaml --seed 42}}
```

- **Hardware**: {{e.g. 2×RTX 3090; large runs need a cluster allocation}}
- **Key versions**: {{e.g. torch 2.4, pyg 2.6}}

## Repository layout

```
{{e.g.
configs/    experiment configs
scripts/    data and utility scripts
src/        model and training code
research/   plan.json / results.json / manuscript.json (read by the dashboard)
figures/    result images
paper/      manuscript source
}}
```

## Logging conventions

> This section is the standing agreement for Claude Code in this repository; humans
> should follow it too. The goal: any experiment should still be fully understandable
> and reproducible a month later.

1. **Commit format**: `<type>: <one-line summary>`, where type is one of
   `exp` (start/run an experiment) · `result` (a conclusion) · `data` · `analysis` ·
   `fig` · `paper` · `feat` · `fix` · `docs` · `chore`.
   A `result:` commit states the key numbers, how they compare to expectation, and the next step.
   *(A project with its own established prefixes should keep them — the dashboard
   renders any `type:` or `type(scope):` prefix and lets you filter on it.)*
2. **Before an experiment**: register it in `research/plan.json` (id, hypothesis,
   expected result, dates) with `status: "planned"`, flipping to `"running"` at launch.
3. **When an experiment finishes** — in the same commit:
   - update `research/plan.json`: set `status` to `done` (or `blocked`, with the reason
     and the recovery condition in `next`), fill `actual` with the real numbers, fill `next`;
   - prepend an entry to the `results` array in `research/results.json`: `summary` states the
     conclusion in one sentence including the numbers; anything that plots goes in `chart`,
     other images go to `figures/` and are referenced from `images`;
   - commit as `result: ...`.
4. **Figures**: saved under `figures/`, named `<experiment-id>_<description>.png`.
5. **Dates** are always `YYYY-MM-DD`. **Conclusions** must distinguish "as expected /
   partly as expected / hypothesis refuted" — a refuted hypothesis is a result and is
   recorded like any other.
6. **Provenance belongs in the record.** Every number in `results.json` should say in
   `notes` where it came from (a metrics CSV, a log file, or this document's prose).
   Numbers that exist only in prose — because a cluster mirror lags, say — are marked so
   they are never mistaken for a reproducible artifact.
7. **When the method or plan changes materially**, update the matching section here in
   the same commit (`docs: ...`).

## research/ data files

The dashboard reads three files from `research/`. They are the structured view of this
document — when the two disagree, this file wins and the JSON is corrected.

**`plan.json`** — experiments and timeline:

```json
{
  "experiments": [
    {
      "id": "E1",
      "name": "Experiment name",
      "status": "planned | running | done | blocked",
      "start": "2026-08-05",
      "end": "2026-08-12",
      "hypothesis": "What is being tested",
      "expected": "Quantified expectation",
      "actual": "Real outcome (filled in on completion)",
      "next": "Next step, or why it is blocked",
      "priority": "high (optional)"
    }
  ],
  "milestones": [{ "date": "2026-09-15", "label": "Paper draft", "done": false }],
  "notes": "Resources, constraints, risks (optional)"
}
```

**`results.json`** — results, newest first:

```json
{
  "results": [
    {
      "id": "R1",
      "experiment": "E1",
      "title": "Result title",
      "date": "2026-08-10",
      "summary": "One-sentence conclusion with the key numbers",
      "metrics": [{ "name": "Test MAE", "value": "0.296", "delta": "-12%", "good": "down" }],
      "chart": {
        "type": "line | bar | scatter",
        "xLabel": "X", "yLabel": "Y",
        "series": [{ "name": "Series", "data": [[1, 0.52], [2, 0.44]] }]
      },
      "images": ["figures/E1_curve.png"],
      "notes": "Provenance and caveats",
      "tags": ["E1"]
    }
  ]
}
```

`chart` and `images` — provide at least one. `data` holds `[x, y]` pairs where x may be a
number or a category label. `good` (`down`/`up`) says which direction is an improvement and
colours the delta. Scatter plots show at most 3 series; `"diag": true` adds a y=x reference line.

**`manuscript.json`** — the paper in progress:

```json
{
  "venue": "Pattern Recognition (Elsevier)",
  "status": "planned | drafting | internal-review | submitted | under-review | revision | accepted | rejected",
  "target": "2026-08-31",
  "source": "paper/main.tex",
  "pdf": "paper/main.pdf",
  "notes": "Framing decisions, known gaps, fallback plan"
}
```

The dashboard fetches `source` and reads it live — no compilation required. It extracts the
title, authors, abstract, section outline with per-section word counts, figures, unique
references and **every unresolved `\todo{}`** (an empty marker still shows the sentence it
interrupts). `.md` sources work too, with `TODO:`/`FIXME:` markers. One level of
`\input{}`/`\include{}` is inlined. Omit `source` before drafting starts and the entry still
tracks venue, status and the countdown to `target`. For several papers, use
`{"manuscripts": [ … ]}`.

## Schedule

| Stage | Window | Goal | Status |
|-------|--------|------|--------|
| {{stage 1}} | {{MM-DD ~ MM-DD}} | {{goal}} | ⏳ not started |
| {{stage 2}} | {{MM-DD ~ MM-DD}} | {{goal}} | ⏳ not started |

## References and related work

- {{key papers and links}}
- {{related codebases}}
