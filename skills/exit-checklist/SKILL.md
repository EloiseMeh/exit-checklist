---
name: exit-checklist
description: Run a full session exit workflow — commit, push, build, deploy, and update docs. Use when finishing a session, wrapping up work, or before closing the terminal.
---

# Exit Checklist

Run this checklist when the user says they're done, about to exit, wrapping up, or asks to run the exit checklist.

Execute each step in order. If a step doesn't apply (e.g. no deploy platform detected), skip it and note why in the final report.

---

## Step 1: Find all git repos

Scan the current working directory and its immediate subdirectories for `.git` folders. Collect every repo path — the rest of the checklist runs on each one.

If no git repos are found, tell the user and stop.

## Step 2: Kill running dev servers

Check for processes listening on common dev ports: 3000, 4321, 5173, 8080, 8000, 8888.

```bash
lsof -i -P -n | grep LISTEN | grep -E ':(3000|4321|5173|8080|8000|8888) '
```

If any are found, list them and ask the user before killing. Kill with `kill <pid>` (not `kill -9`).

## Step 3: Check repo status

For each repo, run:

1. `git status --short` — check for uncommitted changes (do not use `-uall`)
2. `git log @{u}..HEAD --oneline 2>/dev/null` — check for unpushed commits
3. `git rev-parse --abbrev-ref HEAD` — current branch name
4. `git fetch --dry-run 2>&1` — check if behind remote

Report findings before proceeding. Flag any repos on a feature branch with unpushed work.

## Step 4: Commit and push

For each repo with uncommitted changes:

1. Stage relevant files (prefer naming specific files over `git add -A`)
2. Do NOT stage files that look like secrets (`.env`, credentials, tokens)
3. Write a concise commit message summarizing the session's changes
4. Push to the remote tracking branch

If the repo has no remote, skip pushing and note it in the report.

## Step 5: Build

For each repo that has a `package.json` with a `build` script:

```bash
npm run build
```

If the build fails, stop and report the error. Do not deploy broken builds.

## Step 6: Deploy

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

Look for a docs file in this order: `UPDATES.md`, `CHANGELOG.md`. Use whichever exists. If neither exists, skip this step.

Append a session summary section with:
- Today's date
- What was changed (brief bullet points)
- Remaining TODOs (if any)

## Step 8: Final report

Print a summary table in the chat:

```
Exit Checklist Complete
=======================
Repos found:     <count>
Dev servers:     <killed N / none found>
Committed:       <list of repos>
Pushed:          <list of repos>
Built:           <list of repos>
Deployed:        <platform: URL for each>
Docs updated:    <list of files>

Remaining TODOs:
- <any unfinished work or issues encountered>
```

If everything succeeded with no remaining work, end with: "Clean exit — all work saved and deployed."
