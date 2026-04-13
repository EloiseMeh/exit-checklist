# Exit Checklist

A [Claude Code](https://claude.com/claude-code) plugin that runs a full session exit workflow with a single command. No more forgotten commits, broken deploys, or lost context.

## Who is this for?

Any developer using Claude Code who works across sessions and wants a clean handoff every time. Whether you're building a frontend app, a backend API, or a multi-repo project — this plugin makes sure nothing gets left behind when you wrap up.

## What it does

When you run `/exit-checklist`, it walks through a complete shutdown sequence:

1. **Finds all git repos** in your working directory
2. **Kills running dev servers** on common ports (3000, 5173, 8080, etc.)
3. **Checks repo status** — uncommitted changes, unpushed commits, branch state
4. **Commits and pushes** dirty repos with a session summary
5. **Builds** projects that have a build step
6. **Deploys** to your hosting platform (auto-detects Netlify, Cloudflare, or Vercel)
7. **Updates docs** — appends a session summary and remaining TODOs to your changelog
8. **Reports** final status so you know exactly what shipped

## Install

```bash
claude plugin add github:EloiseMeh/exit-checklist
```

## Usage

At the end of your session, either run the command:

```
/exit-checklist
```

Or just use natural language — any of these will trigger the checklist automatically:

- "I'm about to exit"
- "exiting"
- "wrapping up"
- "I'm done"
- "closing up"
- "signing off"
- "good to exit?"
- "can I exit?"

## Supported platforms

The plugin auto-detects your hosting platform:

| Platform | How it's detected | What happens |
|----------|------------------|--------------|
| **Netlify** | `.netlify/` or `netlify.toml` | Runs `netlify deploy --prod` |
| **Cloudflare** | `wrangler.jsonc` or `wrangler.toml` | Confirms git push triggered auto-deploy |
| **Vercel** | `.vercel/` or `vercel.json` | Runs `vercel --prod` |

If you use multiple platforms (e.g. Netlify + Cloudflare), it deploys to all of them.

## Customization

The skill is a plain markdown file at `skills/exit-checklist/SKILL.md`. Fork this repo and edit it to fit your workflow:

- Add or remove steps
- Change which ports to check for dev servers
- Add project-specific deploy commands
- Customize the docs format

## License

MIT
