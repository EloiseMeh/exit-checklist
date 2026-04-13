# Exit Checklist for Claude Code

**One command to wrap up your entire session.** Commit, push, build, deploy, update docs — done.

No more "did I push that?", no more orphaned dev servers, no more undocumented changes.

```
> I'm done for today

Exit Checklist Complete
=======================
Repos found:     3
Dev servers:     killed 1 (port 5173)
Committed:       home, snake, pretzel
Pushed:          home, snake, pretzel
Built:           pretzel
Deployed:        Netlify: 3 sites, Cloudflare: 3 sites
Docs updated:    UPDATES.md

Clean exit — all work saved and deployed.
```

---

## Install

```bash
claude plugin add github:EloiseMeh/exit-checklist
```

## How to use

Run the slash command:

```
/exit-checklist
```

Or just talk naturally — the plugin auto-triggers on phrases like:

> "I'm about to exit" / "wrapping up" / "I'm done" / "signing off" / "exiting"

No special syntax needed. Just say you're done and it handles the rest.

---

## What it does

| Step | What happens |
|------|-------------|
| **Find repos** | Scans your working directory for all git repos |
| **Kill dev servers** | Finds and stops processes on ports 3000, 4321, 5173, 8080, 8000, 8888 |
| **Check status** | Uncommitted changes, unpushed commits, branch state |
| **Commit & push** | Stages, commits with a session summary, pushes to remote |
| **Build** | Runs `npm run build` if a build script exists |
| **Deploy** | Auto-detects your platform and ships it |
| **Update docs** | Appends session summary + TODOs to your changelog |
| **Report** | Prints exactly what shipped and what's left |

Steps that don't apply get skipped — the checklist works for any project, whether it's a full-stack app with CI/CD or a simple folder with docs.

## Supported platforms

Auto-detected from your project files:

| Platform | Detected by | Action |
|----------|------------|--------|
| **Netlify** | `.netlify/` or `netlify.toml` | `netlify deploy --prod` |
| **Cloudflare** | `wrangler.jsonc` or `wrangler.toml` | Confirms push triggered auto-deploy |
| **Vercel** | `.vercel/` or `vercel.json` | `vercel --prod` |

Multiple platforms? It deploys to all of them.

## Make it yours

The entire skill is one markdown file — `skills/exit-checklist/SKILL.md`. Fork this repo and customize:

- Add or remove steps
- Change which ports to scan
- Add project-specific deploy commands
- Adjust the docs format

No config files, no dependencies, no build step. Just markdown.

---

## License

MIT
