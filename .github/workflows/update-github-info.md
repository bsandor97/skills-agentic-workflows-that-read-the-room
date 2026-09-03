---
name: update-github-info
# The copilot engine defaults to claude-sonnet-4, which is not available on the
# Copilot Free tier. 'mini' resolves to the first available small model
# (haiku -> gpt-5-mini -> gpt-5-nano -> gemini-flash-lite).
engine:
  id: copilot
  model: mini
on:
  schedule:
    - cron: '17 9 * * *'  # daily, offset minute to avoid load spikes
  workflow_dispatch: {}
description: |
  Agentic workflow to keep site/content/github-info.md up-to-date by
  reading internal notes and the GitHub Blog, then proposing edits via
  a pull request for Mona to review.
# Tools granted to the agent. edit lets the agent stage file changes, which are
# published through the safe-outputs flow rather than written directly to main.
tools:
  edit:
  web-fetch:
network:
  allowed:
    - github.blog
    - github.com
    - awesome-copilot.github.com
safe-outputs:
  create-pull-request:
    title-prefix: "[mona] "
    draft: true
# Do NOT compile this workflow file in the repository; gh aw compile must be
# run by maintainers when ready. This .md is intentionally a single-file
# agentic workflow that uses safe-outputs to propose changes.
---

## Summary

This agentic workflow is responsible for keeping site/content/github-info.md
up-to-date by combining repository notes and authoritative content from the
GitHub Blog. It runs daily (schedule) and on demand (workflow_dispatch).

## Agent Instructions

When triggered, the agent should perform the following steps in order. Use
GitHub repository API tools to read repository files rather
than local filesystem or CLI commands. Use web-fetch for external guidance.

1. Read notes/mona-notes.md from this repository using the GitHub repository
   API. Do not read the file from the runner filesystem.

2. Web-fetch the following public pages (use the web-fetch tool):
   - <https://github.blog/latest/>
   - <https://github.blog/changelog/>
   - <https://awesome-copilot.github.com/workflows/>

   Summarize any relevant new or changed information about GitHub features,
   product changes, or community guidance that should be reflected in the
   repository's site/content/github-info.md.

3. Combine the repository notes (notes/mona-notes.md) and the fetched blog
   summaries into a concise, factual update for site/content/github-info.md.
   Preserve any important notes or attribution lines from Mona's notes when
   appropriate.

4. Create or update site/content/github-info.md with the composed content.
   Use the edit tool capabilities together with the safe-outputs create-pull-request
   flow so changes are proposed on a branch and opened as a pull request rather
   than being written directly to main.

5. Open a pull request titled "Update GitHub info from GitHub Blog and Mona's notes"
   and request a review from Mona (default reviewer: Mona). In the PR body,
   include a short summary of the sources used and the exact fetched URLs.

6. Attach a short checklist in the PR body for Mona to confirm: "Sources read:
   - notes/mona-notes.md (repo)
   - <https://github.blog/latest/>
   - <https://github.blog/changelog/>
   - <https://awesome-copilot.github.com/workflows/>"

7. Exit and report the created pull request URL in the agent's structured output.

## Validation

Before proposing changes, validate that the repository read operations used the
GitHub repository API and that the web-fetch calls returned HTTP 200 responses.
If any fetch fails, include the status code and URL in the PR body and do not
change site/content/github-info.md.

## Usage

This file is the agentic workflow definition. Do NOT compile it as part of
this change. Maintainers should run `gh aw compile` when ready to create the
.lock.yml file and enable execution in Actions.

---

<!-- End of workflow -->
