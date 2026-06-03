# vllm-issue-scout

An hourly GitHub Actions cron that polls [`vllm-project/vllm`](https://github.com/vllm-project/vllm)
for **fresh, unclaimed, CPU-verifiable** issues, applies a duplicate-work checklist,
and opens a report issue here when it finds clean candidates — so they can be picked
up before the swarm.

## How it works
- Runs at **:17 past every hour** (`.github/workflows/scout.yml`; also `workflow_dispatch`).
- Looks at upstream issues created in the last ~70 minutes that are **unassigned** and
  have **≤2 comments**.
- Filters out GPU/kernel/model-specific issues (not verifiable on a CPU-only Mac/container),
  checking the **title and the issue body** (with the `collect_env` dump stripped first).
- **Dedup** (the part that matters): skips any issue that already has an open PR — checked
  by **issue-number** *and* **keyword/area** search, because existing fixes often predate
  the issue and don't cite its number.
- **Self-dedup:** skips issues already reported in a prior run (matched against existing
  report-issue bodies), so the look-back overlap doesn't create duplicate reports.

## Output
When candidates are found it opens an issue here titled `vLLM scout — N fresh candidate(s) (…)`,
listing each candidate with its dedup evidence. Results always appear in the Actions run
**Job Summary** too. Each report still says *confirm keyword/area dedupe + root-cause-in-core
before starting* — the cron narrows the field; a human/agent makes the final call.
