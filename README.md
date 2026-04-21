# dangaiden's Claude Code Skills

Custom [Claude Code](https://claude.ai/code) skills by [@dangaiden](https://github.com/dangaiden).

## Skills

### `/dangaiden-project-overview`

Generates a concise overview of any project — structure, purpose, recent git activity, entry points, and open TODOs.

**Usage:**
```
/dangaiden-project-overview              # full overview
/dangaiden-project-overview backend      # focus on a specific area
```

## Installation

### Claude Code

```
/plugin marketplace add dangaiden/something-skill
/plugin install something-skill@something-skill
/reload-plugins
```

The skill will appear as `/something-skill:dangaiden-project-overview`.

### Other agents (Cursor, Copilot, etc.)

```bash
npx skills add dangaiden/something-skill
```

> **Note:** `npx skills add` installs into `.agents/skills/`, which Claude Code does not read. Use the plugin marketplace flow above for Claude Code.

## License

MIT
