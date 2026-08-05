# Research Log

A single-file HTML dashboard that pulls a research project's state straight from the GitHub
API — no build step, no dependencies, no server. Open `index.html` locally or deploy it to
GitHub Pages.

| Tab | What it shows | Source |
| --- | --- | --- |
| Overview | Per-project cards: commit activity, experiment status counts, latest result | everything below |
| Commits | Commit timeline grouped by day, with type chips, search and filters | GitHub commits API |
| Results | Metric tiles + line/bar/scatter charts (hover readouts, data-table view) + images | `research/results.json` |
| Methods | CLAUDE.md rendered as a readable document with a table of contents | `CLAUDE.md` |
| Plan | Status tiles, Gantt timeline, milestones, hypothesis/expected/actual cards | `research/plan.json` |
| Manuscript | Title, abstract, section outline with word counts, figures, references, open TODOs | `research/manuscript.json` + the paper source |

## Quick start

1. **Open** `index.html` (double-click works — the GitHub API allows cross-origin requests, so
   no local server is needed). It starts on a built-in demo project so every feature is visible.
2. **Link your repositories**: press ⚙ and enter `owner/repo` per line (`owner/repo#branch` to
   pin a branch), or type a GitHub username to pick from a list.

   You can also pass them in the URL, which is handy across devices:
   `…/research-dashboard/?repos=owner/repo1,owner/repo2#results`
   The list is written to that device's localStorage, so later visits need no parameter.
   **Prefer this for private projects** — the list lives only in your link and your browser,
   never in the public Pages repository.
3. **Private repositories / higher rate limit** (optional): create a read-only token under
   GitHub → Settings → Developer settings → Fine-grained tokens (repository permission
   **Contents: Read-only**) and paste it into ⚙. Without a token, public repositories work at
   60 requests/hour; a full project load costs about 5. The token is stored only in your browser.

## Connecting a research project

Add three things to the project repository:

1. **`CLAUDE.md`** at the root — copy [templates/CLAUDE-template.md](templates/CLAUDE-template.md)
   and fill it in. It doubles as the working brief for Claude Code and as the source for the
   Methods tab.
2. **`research/plan.json`** — experiments, hypotheses, expected results, timeline.
   See [templates/plan.example.json](templates/plan.example.json).
3. **`research/results.json`** — results with chart data, metrics and conclusions.
   See [templates/results.example.json](templates/results.example.json). Images go in `figures/`
   and are referenced from a result's `images` field.

Optionally add **`research/manuscript.json`**
([template](templates/manuscript.example.json)) pointing at the paper source, and the
Manuscript tab reads the LaTeX or Markdown live.

The day-to-day loop — the template's "Logging conventions" section states these as a standing
agreement for Claude Code, which then maintains the files after each experiment:

```text
experiment starts   → plan.json entry flips to running        → commit "exp: ..."
experiment finishes → plan.json status + actual; results.json → commit "result: <conclusion + numbers>"
```

Commit prefixes (`exp:` `result:` `data:` `analysis:` `fig:` `paper:` `feat:` `fix:` `docs:`
`chore:`) render as type chips on the Commits tab. `type(scope):` and project-specific
prefixes (`stage2:`, `sync:`, `downstream:`) also work — unknown prefixes get a neutral chip
and remain filterable, so a project never has to change its existing commit habits for the
dashboard.

## The Manuscript tab

`research/manuscript.json` describes the paper; everything else is derived from the source
file at load time, so nothing has to be compiled or kept in sync by hand:

```json
{
  "venue": "Pattern Recognition (Elsevier)",
  "status": "drafting",
  "target": "2026-08-31",
  "source": "paper/main.tex",
  "pdf": "paper/main.pdf",
  "notes": "Framing decisions, known gaps, fallback plan"
}
```

The page shows the title and authors, a countdown to `target`, an abstract, per-section word
counts (parents roll up their subsections, so an unwritten stub reads as 0), figure and
reference inventories, and **every unresolved `\todo{}`** — including empty markers, which are
shown together with the sentence they interrupt. `.md` manuscripts work the same way using
`TODO:`/`FIXME:` markers. One level of `\input{}`/`\include{}` is inlined. Drop `source`
before drafting starts and the entry still tracks venue, status and deadline. Several papers:
`{"manuscripts": [ … ]}`.

Word counts are approximate: LaTeX markup is stripped heuristically, so treat them as a
progress signal rather than a submission-ready count.

## Deploying to GitHub Pages (optional)

Opening the file locally is already fully functional; deploy when you want a stable URL:

```bash
cd research-dashboard
git init -b main   # if not already a repository
git add -A && git commit -m "feat: research log"
# create a repository on github.com, then:
git remote add origin https://github.com/<user>/research-dashboard.git
git push -u origin main
```

In the repository, Settings → Pages → Source → **GitHub Actions**
([.github/workflows/pages.yml](.github/workflows/pages.yml) is included; you select the source
once and every later `git push` deploys automatically). The site appears at
`https://<user>.github.io/research-dashboard/`.

> A Pages site is public on free accounts even when its repository is private. The token never
> leaves the browser, but if you would rather not expose the page itself, use it locally only.
>
> **Do not put unpublished projects' repository names in a deployed `projects.json`** — that
> file ships with the public site and would disclose what you are working on before submission.
> Use the `?repos=` link or ⚙ instead; both stay in the browser. `projects.json` is for public
> projects only.

## Field reference

Full details are in the "research/ data files" section of
[templates/CLAUDE-template.md](templates/CLAUDE-template.md). Key points:

- `chart.type`: `line` / `bar` / `scatter`; `series[].data` holds `[x, y]` pairs, x numeric or categorical
- Scatter shows at most 3 series (a colour-vision-safety limit); `"diag": true` adds a y=x line
- `metrics[].delta` plus `good` (`down`/`up`) drives the up/down colouring; new results go at the **front** of the array
- Dates are always `YYYY-MM-DD`

## Troubleshooting

- **Stuck on "Loading…" or a request failure** — check connectivity; `file://` is fine, the API allows it.
- **Rate limit (403)** — 60 requests/hour without a token, 5,000 with one.
- **404 on a private repository** — the token needs Contents read access to that repository.
- **Raw HTML in CLAUDE.md does not render** — the renderer supports Markdown only and escapes HTML.
- **A chart is missing** — check that `research/results.json` is valid JSON (the page reports parse errors) and that `chart.series` is non-empty.
- **The manuscript tab says the source failed to load** — check the `source` path in `manuscript.json`; it is relative to the repository root.
- **Repository updated but the page has not changed** — press ⟳ (data is cached for the session).
