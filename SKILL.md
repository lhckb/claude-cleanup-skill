---
name: cleanup
description: Deep clean macOS caches, logs, and unnecessary files to free disk space. Safe cleanup only — never deletes configs, user data, or important files. Always scans and shows candidates before deleting anything; never deletes without explicit user confirmation.
disable-model-invocation: true
---

# macOS Deep Cleanup

Scan for cleanup candidates on macOS, present them to the user with sizes, and **only delete after the user explicitly approves**. Safety first — only ever delete expendable caches, never configs or user data, and never without a greenlight.

## Step 1: Pre-scan — show current disk usage

Run `df -h /` to show available disk space before cleanup.

## Step 2: Scan only — measure every candidate, delete nothing

For each category below, measure the size that *would* be freed using a non-destructive command (e.g. `du -sh`), suppressing errors for paths that don't exist. Do **not** run any deletion, `cache clean`, `cleanup`, `prune`, or `flushcache` command in this step — this step is read-only.

### User Caches
`~/Library/Caches/*`

### User Logs
`~/Library/Logs/*`

### npm / npx cache
`~/.npm/_cacache`, `~/.npm/_npx/*`, `~/.npm/_logs/*` (and what `npm cache verify` reports as reclaimable)

### Homebrew cleanup
What `brew cleanup --prune=all --dry-run` reports it would remove

### pip cache
What `pip3 cache info` reports as cache size

### VS Code caches
`~/Library/Application Support/Code/Cache/*`, `CachedData/*`, `CachedExtensions/*`, `CachedExtensionVSIXs/*`, `Code Cache/*`

### Cursor caches (same structure as VS Code)
`~/Library/Application Support/Cursor/Cache/*`, `CachedData/*`, `CachedExtensions/*`, `Code Cache/*`

### Claude Desktop app caches
`~/Library/Application Support/Claude/vm_bundles/*`, `Cache/*`, `GPUCache/*`, `Code Cache/*`, `Service Worker/CacheStorage/*`

### Chrome caches (NOT profile data, NOT extensions, NOT bookmarks)
`~/Library/Application Support/Google/Chrome/*/Service Worker/CacheStorage/*`, `*/Application Cache/*`, `*/GPUCache/*`, `GrShaderCache/*`, `ShaderCache/*`

### Spotify cache
`~/Library/Application Support/Spotify/PersistentCache/*`

### Slack cache
`~/Library/Application Support/Slack/Cache/*`, `Code Cache/*`, `GPUCache/*`, `Service Worker/CacheStorage/*`

### Discord cache
`~/Library/Application Support/discord/Cache/*`, `Code Cache/*`, `GPUCache/*`

### Zoom cache
`~/Library/Application Support/zoom.us/AutoUpdater/*`

### Docker (only report if daemon is running; skip entirely otherwise)
What `docker system df` reports as reclaimable (dangling images, build cache)

### macOS system caches (requires sudo — report as "requires sudo", skip measuring if unavailable)
`/Library/Caches/*`, `/private/var/log/asl/*.asl`

### DNS cache
Not a disk-space item — note separately that a DNS flush is available, not part of the size table.

### .DS_Store files (home directory)
Count and total size of `.DS_Store` files via `find ~ -name ".DS_Store" -maxdepth 4`

## Step 3: Present candidates and get explicit approval

Show the user a table: Category | Path(s) | Size. Include a grand total of reclaimable space. Explicitly ask the user to confirm before deleting anything — e.g. "Delete all of the above?" or let them pick a subset by category (e.g. "just npm and Homebrew").

**Do not proceed to Step 4 under any circumstances until the user responds with an explicit go-ahead in this turn.** A prior approval, a general instruction to "clean up", or running `/cleanup` itself is not approval for this step — the scan-then-ask happens every run. If the user says to skip or exclude a category, drop it from the delete list. If the user doesn't respond or declines, stop here and change nothing.

## Step 4: Execute only the approved deletions

Only after explicit confirmation, run the deletion/cleanup command for each **approved** category (and only those), suppressing errors for paths that don't exist. These are the commands corresponding to each category above:

```bash
rm -rf ~/Library/Caches/* 2>/dev/null
rm -rf ~/Library/Logs/* 2>/dev/null
npm cache clean --force 2>/dev/null
rm -rf ~/.npm/_npx/* 2>/dev/null
rm -rf ~/.npm/_logs/* 2>/dev/null
brew cleanup --prune=all 2>/dev/null
pip3 cache purge 2>/dev/null
rm -rf ~/Library/Application\ Support/Code/Cache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Code/CachedData/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Code/CachedExtensions/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Code/CachedExtensionVSIXs/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Code/Code\ Cache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Cursor/Cache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Cursor/CachedData/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Cursor/CachedExtensions/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Cursor/Code\ Cache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Claude/vm_bundles/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Claude/Cache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Claude/GPUCache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Claude/Code\ Cache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Claude/Service\ Worker/CacheStorage/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Google/Chrome/Default/Service\ Worker/CacheStorage/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Google/Chrome/*/Service\ Worker/CacheStorage/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Google/Chrome/Default/Application\ Cache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Google/Chrome/Default/GPUCache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Google/Chrome/GrShaderCache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Google/Chrome/ShaderCache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Spotify/PersistentCache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Slack/Cache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Slack/Code\ Cache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Slack/GPUCache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/Slack/Service\ Worker/CacheStorage/* 2>/dev/null
rm -rf ~/Library/Application\ Support/discord/Cache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/discord/Code\ Cache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/discord/GPUCache/* 2>/dev/null
rm -rf ~/Library/Application\ Support/zoom.us/AutoUpdater/* 2>/dev/null
docker system prune -f 2>/dev/null
sudo rm -rf /Library/Caches/* 2>/dev/null
sudo rm -rf /private/var/log/asl/*.asl 2>/dev/null
find ~ -name ".DS_Store" -maxdepth 4 -delete 2>/dev/null
```

The DNS flush (`sudo dscacheutil -flushcache && sudo killall -HUP mDNSResponder`) frees no disk space and is not part of the candidate table — only run it if the user separately asks for it.

Track bytes actually freed per approved category as you go.

## Step 5: Deep scan for large space hogs not already covered

After the approved deletions run, do a quick scan for any other large cache-like directories that may have been missed:

```bash
du -sh ~/Library/Application\ Support/*/Cache* ~/Library/Application\ Support/*/GPUCache ~/Library/Application\ Support/*/Service\ Worker 2>/dev/null | sort -rh | head -10
```

If any are over 500 MB, add them to a *new* candidate list and ask the user before deleting — do not delete these in the same approval as Step 3.

## Step 6: Post-scan — show results

Run `df -h /` again and calculate total space freed. Present a summary table showing:
- Space freed per approved category
- Total space freed
- Current available disk space

## SAFETY RULES — DO NOT VIOLATE

- NEVER delete or clean anything before the user has seen the full candidate list and explicitly approved it in this run
- NEVER treat a prior run's approval, or the act of invoking `/cleanup`, as approval — ask again every time
- NEVER delete anything under ~/Documents, ~/Desktop, ~/Downloads, ~/Work, ~/Projects, ~/Code
- NEVER delete config files (anything named config.json, .env, preferences, settings, profiles, bookmarks, etc.)
- NEVER delete ~/Library/Application Support/*/Local Storage (contains app state)
- NEVER delete ~/Library/Application Support/*/IndexedDB (contains app data)
- NEVER delete ~/Library/Application Support/*/databases
- NEVER delete ~/Library/Keychains or anything auth/credential related
- NEVER delete SSH keys, GPG keys, or tokens
- NEVER delete git repositories or .git directories
- NEVER delete node_modules (user may need them for active projects)
- NEVER delete nvm Node versions (user manages these manually)
- ONLY delete directories explicitly listed above, and only the subset the user approved — do not improvise
- If unsure whether something is safe, ASK the user first
