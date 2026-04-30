# Claude Code Skill Updates — User Guide

## Two Distribution Paths

### 1. Claude Code Plugin
Installed via `/plugin` from a marketplace. Managed by Claude Code itself — versioned through `plugin.json` (`version` field, falls back to git commit SHA).

### 2. NPX Installer
Standalone install via `npx <package>`. **Not** managed by Claude Code — behaves like any other npm package.

---

## Auto-Update Behavior

| Distribution | Default | Notes |
|---|---|---|
| Official Anthropic marketplaces | **ON** | Refreshes on Claude Code startup |
| Third-party marketplaces (this repo) | **OFF** | Users must opt in or refresh manually |
| NPX | **N/A** | Re-run the install command to update |

When a plugin update applies, users see a notification prompting them to run `/reload-plugins` (no CLI restart needed).

---

## Forcing a Refresh

### Plugin users
```
/plugin marketplace update <marketplace-name>   # refresh the catalog
/plugin update <plugin-name>                    # pull the latest version
/reload-plugins                                 # reload without restarting
```

Nuclear option — clear the cache:
```bash
rm -rf ~/.claude/plugins/cache
```

Useful environment variables:
- `DISABLE_AUTOUPDATER=1` — disables Claude Code self-update
- `FORCE_AUTOUPDATE_PLUGINS=1` — forces plugin updates even if self-update is off

### NPX users
```bash
npx <package>@latest
```

---

## Tips for Maintainers

- Bump the `version` field in `plugin.json` on each release — gives users a clean signal instead of relying on commit SHAs.
- Document the refresh commands in your README so users know how to pull updates.
- Since third-party marketplaces don't auto-update by default, suggest users enable auto-update in their settings if they want latest-always.
