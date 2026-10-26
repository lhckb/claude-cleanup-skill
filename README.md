# `/cleanup` — macOS Deep Clean for Claude Code

> Scans first. Shows you everything. Deletes nothing until you say go.

Your Mac accumulates **gigabytes** of cache junk from browsers, editors, package managers, and apps you forgot you had. This Claude Code skill finds it all, shows you a full breakdown with sizes, and only deletes what you explicitly approve.

## What it cleans

| Category | What gets nuked |
|----------|----------------|
| **System** | `~/Library/Caches/*`, `~/Library/Logs/*`, system caches, `.DS_Store` files, DNS cache |
| **Package managers** | npm, npx, pip, Homebrew (old versions + cache) |
| **Editors** | VS Code, Cursor — cached data, extensions cache, code cache |
| **AI tools** | Claude Desktop — VM bundles, GPU cache, service workers |
| **Browsers** | Chrome — service workers, GPU/shader cache (never touches bookmarks, history, or extensions) |
| **Chat apps** | Slack, Discord, Zoom — cache and auto-updater bloat |
| **Media** | Spotify persistent cache |
| **Containers** | Docker dangling images and build cache (only if daemon is idle) |

Every run scans every category first and shows you a candidate table with sizes — nothing is deleted until you approve it (all of it, or just a subset). After the approved deletions run, it deep-scans for remaining space hogs over 500 MB and asks separately before touching those too.

## What it NEVER touches

- Your files — Documents, Desktop, Downloads, Work, Projects, Code
- App configs — settings, preferences, profiles, bookmarks
- App state — Local Storage, IndexedDB, databases
- Credentials — Keychain, SSH keys, GPG keys, tokens, `.env` files
- Dev environments — git repos, `node_modules`, nvm versions
- Anything not on the explicit allow-list — **it doesn't improvise**
- Anything you didn't explicitly approve — **it never deletes before asking**

## Example output

```
📊 Disk before: 42 GB available of 460 GB

🔍 Scanning cleanup candidates (nothing deleted yet)...

| Category           | Reclaimable |
|--------------------|-------------|
| User Caches        | 3.2 GB      |
| User Logs          | 180 MB      |
| npm / npx          | 420 MB      |
| Homebrew           | 890 MB      |
| pip                | 65 MB       |
| VS Code            | 310 MB      |
| Cursor             | 275 MB      |
| Claude Desktop     | 140 MB      |
| Chrome             | 520 MB      |
| Spotify            | 1.1 GB      |
| Slack              | 380 MB      |
| Discord            | 210 MB      |
| Docker             | 2.4 GB      |
| System caches      | 450 MB      |
| .DS_Store files    | 2 MB        |

Total reclaimable: 10.5 GB

Delete all of the above? (or name the categories to skip)

> yes

🧹 Cleaning approved categories...

✅ Total freed: 10.5 GB
📊 Disk after: 52.5 GB available of 460 GB
```

## Install

**One-liner (recommended):**

```bash
mkdir -p ~/.claude/skills/cleanup && curl -fsSL https://raw.githubusercontent.com/dancolta/claude-cleanup-skill/main/SKILL.md -o ~/.claude/skills/cleanup/SKILL.md
```

**Or clone:**

```bash
git clone https://github.com/dancolta/claude-cleanup-skill.git
mkdir -p ~/.claude/skills/cleanup
cp claude-cleanup-skill/SKILL.md ~/.claude/skills/cleanup/SKILL.md
```

**Or project-level** — drop `SKILL.md` into `.claude/skills/cleanup/` to share with your team via git.

## Usage

```
/cleanup
```

That's it. Typical results: **2–15 GB freed**.

## Requirements

- macOS (tested on Sonoma, Sequoia)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Optional: `sudo` for system-level caches and DNS flush (skipped gracefully if unavailable)

## Customization

It's just a markdown file — fork and add your own targets (Xcode DerivedData, JetBrains, Android Studio, Conda, Yarn/pnpm). Follow the same pattern: explicit paths, `2>/dev/null`, and add to the safety rules if needed.

## How it works

Claude Code skills are markdown instructions. When you type `/cleanup`, Claude reads the `SKILL.md` and executes each step in your terminal. No binaries, no background processes, fully transparent.

## License

MIT

## Contributing

Found a cache worth cleaning? Open a PR. Rules: only expendable caches, never user data, never configs.

---

> Made by [NodeSparks](https://www.nodesparks.com) — [custom AI tools & automations](https://www.nodesparks.com) that replace the SaaS subscriptions eating your team's time.
