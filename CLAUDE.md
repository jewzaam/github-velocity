# Velocity Dashboard

PR velocity dashboard for GitHub repositories. Self-contained HTML + Python refresh script. Shows merged, opened, and closed-not-merged PRs over a configurable time window with daily and weekly breakdowns. Supports multiple repos aggregated into a single view.

## What This Does

Visualizes PR activity across multiple GitHub repos in four states:
- **Merged → main** (blue) — code landing on the default branch
- **Merged → feature branch** (purple) — work-in-progress landing on integration branches
- **Opened** (green) — new PRs created (still open)
- **Closed without merge** (red) — abandoned, superseded, or reworked PRs

The dashboard shows separate panels for each metric (PR counts, lines changed, contributors, main merges) in both weekly and daily views, plus summary stats, contributor breakdown, longest-lived PRs, and active feature branches.

## Architecture

```
velocity/
  index.html     — dashboard template (fetches data.json for dev, or used as inline template)
  refresh.py     — calls gh CLI, inlines data into index.html, writes output
output/
  dashboard.html — generated self-contained dashboard (gitignored)
CLAUDE.md        — this file
```

### Data Flow

1. `velocity/refresh.py` iterates over repos specified via `--repo` flags
2. For each repo, calls `gh pr list` for merged/open/closed PRs, then enriches each merged PR with commit and review data
3. Compacts each PR to minimal fields, adds repo identifier, main-merge count, and reviewer list
4. Reads `velocity/index.html` as a template and inlines the JSON data
5. Writes self-contained `output/dashboard.html` — open directly in any browser

### Output

The default output is a self-contained HTML file (`output/dashboard.html`) with all data inlined. Open it directly in any browser — no server needed. The template (`velocity/index.html`) can still be served with a local HTTP server alongside `data.json` for development.

```bash
python3 velocity/refresh.py --repo owner/repo                     # single repo
python3 velocity/refresh.py --repo org/a --repo org/b             # multiple repos
python3 velocity/refresh.py --repo org/a --repo org/b --days 14   # different time window
# open output/dashboard.html in any browser — no server needed
```

## data.json Schema

```json
{
  "repos": ["owner/repo1", "owner/repo2"],
  "generated": "2026-04-10T12:40",
  "days": 42,
  "merged": [
    {
      "additions": 1234,
      "deletions": 56,
      "author": {"is_bot": false, "login": "username"},
      "baseRefName": "main",
      "createdAt": "2026-04-01T10:00:00Z",
      "mergedAt": "2026-04-02T14:00:00Z",
      "number": 123,
      "title": "feat: short title",
      "repo": "owner/repo1",
      "mainMerges": 3,
      "reviewers": ["reviewer1", "reviewer2"]
    }
  ],
  "open": [
    {
      "n": 456,
      "t": "feat: short title",
      "a": "username",
      "b": "main",
      "c": "2026-04-05",
      "l": 1290,
      "d": 0,
      "r": "owner/repo1"
    }
  ],
  "closed": [
    {
      "n": 789,
      "t": "superseded by #790",
      "a": "username",
      "c": "2026-04-01",
      "x": "2026-04-03",
      "l": 5000,
      "r": "owner/repo1"
    }
  ]
}
```

Field key for compact open/closed entries:
- `n` = PR number
- `t` = title (truncated)
- `a` = author login
- `b` = base branch (open only)
- `c` = created date
- `x` = closed date (closed only)
- `l` = lines changed (additions + deletions)
- `d` = draft (1/0, open only)
- `r` = repo (owner/repo)

Additional merged PR fields:
- `repo` = source repo (owner/repo)
- `mainMerges` = count of commits merging main into the PR branch (GitHub "Update branch" or manual `git merge main`)
- `reviewers` = sorted list of unique reviewer logins

## Dashboard Panels

1. **N-Week Summary** — human PRs merged, unique contributors, lines changed, avg days to merge, open count, closed-not-merged count, main merges into PRs
2. **Feature vs Main** — merge counts and lines split by target branch, feature branch ratio
3. **PR Counts — Weekly** — 4-color stacked bars per week: merged→main, merged→feature, opened, closed. Median merge age
4. **PR Counts — Daily** — same 4-color bars per day for the full time window. Weekend rows highlighted
5. **Lines Changed — Weekly** — additions (green) and deletions (red) stacked bars per week, with net change
6. **Lines Changed — Daily** — same per day with weekend highlighting
7. **Contributors — Weekly** — unique contributor count per week with bar and name list. Includes PR authors and reviewers
8. **Contributors — Daily** — same per day with weekend highlighting
9. **Main Merges — Weekly** — count of times PRs merged the main branch per week (update-branch / git merge main events)
10. **Main Merges — Daily** — same per day with weekend highlighting
11. **Top Contributors** — ranked by PR count with bar chart, lines changed, avg days open, main vs feature split
12. **Longest-Lived Merged** — top 5 PRs by time from creation to merge
13. **Feature Branches** — list of active feature branches receiving merges

## Dependencies

- `gh` CLI — authenticated with access to the target repos
- Python 3.10+ — for refresh.py (stdlib only, no pip dependencies)
- Modern browser — for the dashboard (uses ES6, Set, fetch)

## Origin

Built to track engineering velocity across multiple teams working on parallel feature branches. The feature-branch vs main split is critical when teams develop on long-lived feature branches and periodically merge to main — tracking only main merges misses most of the daily activity.

The closed-not-merged view surfaces iteration churn — PRs that were superseded, reworked, or abandoned. High churn indicates either healthy iteration (fast feedback loops) or wasted effort (scope changes, miscommunication). The data tells you which.

## Future Direction

- **Per-repo breakdown** — show velocity split by repo within the aggregated dashboard
- **Trend history** — store snapshots over time, show velocity trends (is the team accelerating or decelerating?)
- **PR-to-issue correlation** — map PRs to issue tracker features. Show velocity per feature, not just per repo
- **Auto-refresh** — cron integration to regenerate dashboard on a schedule
- **Cost of closed PRs** — quantify wasted effort: lines written in closed-not-merged PRs as a percentage of total lines. High ratios suggest process problems.

## Code Style

- Vanilla JS, no frameworks, no build step
- Dark theme (#0f1214 background, #1a1d21 cards)
- CSS is inline in the HTML head
- Template fetches data.json for dev; refresh.py inlines data for distribution
- refresh.py uses only stdlib (subprocess, json, argparse, datetime)
