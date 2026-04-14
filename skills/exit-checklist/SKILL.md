---
name: exit-checklist
description: Run a full session exit workflow — commit, push, build, deploy, and update docs. Trigger phrases include "about to exit", "exiting", "wrapping up", "I'm done", "closing up", "signing off", "end of session", "good to exit?", "can I exit?", or any variation indicating the user is finishing their session.
---

# Exit Checklist

**Auto-trigger:** Run this checklist whenever the user signals they are finishing their session. This includes phrases like "about to exit", "exiting", "I'm done", "wrapping up", "closing up", "signing off", "good to exit?", "can I exit?", or similar. Do not wait for the user to explicitly say `/exit-checklist` — natural language triggers should work the same way.

Execute each step in order. **If a step doesn't apply, skip it and note why in the final report — do not stop the checklist early.** Not every project uses git, has a build step, or deploys to a platform. The checklist should always run to completion.

---

## Step 0: Preflight check

Before running the checklist, verify tools are available. Run a single command:

```bash
which git && which netlify; which vercel; which wrangler
```

If a tool is missing, don't error out — just note it and skip steps that need it:
- **No git:** "Git not installed — skipping save steps. Learn more: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git"
- **No netlify/vercel/wrangler CLI:** Skip deploy for that platform silently. It's fine — the user may deploy another way.

If git is missing, skip directly to Step 2 and then Step 7-8.

## Step 1: Find all git repos

Scan the current working directory and its immediate subdirectories for `.git` folders. Collect every repo path.

If no git repos are found, note "No git repos found — skipping git and deploy steps" and continue to Step 2. Steps 3-6 will be skipped automatically.

## Step 2: Kill running dev servers

Check for processes listening on common dev ports: 3000, 4321, 5173, 8080, 8000, 8888.

```bash
lsof -i -P -n | grep LISTEN | grep -E ':(3000|4321|5173|8080|8000|8888) '
```

If any are found, list them and ask the user before killing. Kill with `kill <pid>` (not `kill -9`).

## Step 3: Check repo status

For each repo, run all checks in a single command to save tokens:

```bash
git status --short && git log @{u}..HEAD --oneline 2>/dev/null && git rev-parse --abbrev-ref HEAD && git fetch --dry-run 2>&1
```

This checks: uncommitted changes, unpushed commits, current branch, and whether it's behind remote — all in one call per repo.

Report findings before proceeding. Flag any repos on a feature branch with unpushed work.

**If a repo has no uncommitted changes AND no unpushed commits, mark it as "clean" and skip Steps 4-6 for that repo.** Don't waste time building and deploying code that hasn't changed.

**Classify changes as "code" or "docs-only".** If the only changed files are documentation (`.md`, `usability-test/`, `docs/`, `README`, `CHANGELOG`, `UPDATES`, `LICENSE`), mark the repo as "docs-only". Docs-only repos should be committed and pushed in Step 4, but skip Steps 5-6 (build and deploy) — docs don't affect the live app.

## Step 4: Commit and push

For each repo with uncommitted changes:

1. Check `git config user.name` — if not set, warn: "Git user not configured. Run `git config --global user.name 'Your Name'` and `git config --global user.email 'you@example.com'` to fix." Then skip committing for that repo.
2. Stage relevant files (prefer naming specific files over `git add -A`)
3. Do NOT stage files that look like secrets (`.env`, credentials, tokens)
4. Write a concise commit message summarizing the session's changes
5. Push to the remote tracking branch

If the repo has no remote, skip pushing and note: "No remote configured — changes saved locally only."

## Step 5: Build

**Only for repos that had code changes committed in Step 4. Skip docs-only repos.**

For each changed repo that has a `package.json` with a `build` script:

```bash
npm run build
```

If the build fails, stop and report the error. Do not deploy broken builds.

## Step 6: Deploy

**Only for repos that had code changes committed in Step 4. Skip docs-only repos.**

Auto-detect the hosting platform and deploy:

| Platform | Detection | Deploy command |
|----------|-----------|----------------|
| Netlify | `.netlify/` dir or `netlify.toml` exists | `netlify deploy --prod --dir=<build_output>` |
| Cloudflare | `wrangler.jsonc` or `wrangler.toml` exists | Git push already triggers auto-deploy — just confirm push succeeded |
| Vercel | `.vercel/` dir or `vercel.json` exists | `vercel --prod` |

If multiple platforms are detected (e.g. both Netlify and Cloudflare), deploy to all of them.

For the deploy directory:
- If a `dist/` or `build/` folder exists after the build step, use that
- For static sites with no build step, use `.` (current directory)

## Step 7: Update docs

**Skip this step if all repos were clean and nothing was committed.** There's nothing to document.

Look for a docs file in this order: `UPDATES.md`, `CHANGELOG.md`, `CHANGES.md`. Use the first one found.

If none exists, ask the user: "No changelog found. Want me to create an UPDATES.md to track session changes?" If they say yes, create it. If no, skip this step.

Append a session summary section with:
- Today's date
- What was changed (brief bullet points)
- Remaining TODOs (if any)

## Step 8: Final report

Print a summary table in the chat:

```
Exit Checklist Complete
=======================
Repos found:     <count or "none">
Dev servers:     <killed N / none found>
Committed:       <list of repos or "skipped — no repos"> (note docs-only repos separately)
Pushed:          <list of repos or "skipped — no repos">
Built:           <list of repos or "skipped — no code changes">
Deployed:        <platform: URL for each, or "skipped — no code changes">
Docs updated:    <list of files or "skipped — no changelog found">

Remaining TODOs:
- <any unfinished work or issues encountered>
```

If everything succeeded with no remaining work, end with: "Clean exit — all work saved and deployed."
