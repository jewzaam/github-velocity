# github-velocity

PR velocity dashboard for GitHub repositories. Shows merged, opened, and closed-not-merged PRs over a configurable time window with daily and weekly breakdowns. Supports multiple repos aggregated into a single view.

## Overview

Visualizes PR activity across multiple GitHub repos in four states:

- **Merged to main** — code landing on the default branch
- **Merged to feature branch** — work-in-progress landing on integration branches
- **Opened** — new PRs created (still open)
- **Closed without merge** — abandoned, superseded, or reworked PRs

The dashboard renders separate panels for each metric (PR counts, lines changed, contributors, main merges) in both weekly and daily views, plus summary stats, contributor breakdown, longest-lived PRs, and active feature branches.

## Usage

```bash
python3 velocity/refresh.py --repo owner/repo                     # single repo
python3 velocity/refresh.py --repo org/a --repo org/b             # multiple repos
python3 velocity/refresh.py --repo org/a --repo org/b --days 14   # different time window
python3 velocity/refresh.py --repo org/a --open                   # generate and open in browser
```

### Options

| Option | Description |
|--------|-------------|
| `--repo` | GitHub repo in `owner/repo` format (required, repeatable) |
| `--days` | Time window in days (default: `42`) |
| `--output` | Output file path (default: `output/dashboard.html`) |
| `--open` | Open dashboard in browser after generating |

The output is a self-contained HTML file with all data inlined. Open it directly in any browser, email it, or drop it in Slack — no server required.

### Dependencies

- `gh` CLI — authenticated with access to the target repos
- Python 3.10+ — stdlib only, no pip dependencies
- Modern browser — for the dashboard (ES6, fetch)

## Development

```bash
make install-dev    # create venv and install dev deps
make check          # run format, lint, typecheck, test
make help           # show all available targets
```
