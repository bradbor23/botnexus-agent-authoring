---
name: botnexus-agent-config-entry
description: Use when registering or editing a BotNexus agent in config.json — the object under the top-level "agents" key, keyed by the agent id. Covers every field (provider, model, displayName, enabled, description, isolationStrategy, thinking, contextWindow, toolIds, memory, soul, extensions) with valid values and a complete worked example. Pair with authoring-botnexus-agent-files, which writes the agent's markdown.
license: MIT
compatibility: BotNexus gateway config at ~/.botnexus/config.json. Restart the gateway after editing.
allowed-tools: read, edit
disable-model-invocation: false
metadata:
  category: agent-authoring
---

# Registering a BotNexus agent in config.json

Agents are defined in `~/.botnexus/config.json` under the top-level `agents` object,
**keyed by the agent id**. The id is the key, matches the agent's directory name
(`~/.botnexus/agents/<id>/`), and is stable, lowercase kebab-case.

```jsonc
{
  "agents": {
    "<id>": { /* the agent definition — fields below */ }
  }
}
```

Edit surgically: `config.json` also holds the gateway's own settings and secrets. Add or
change one agent object under `agents`; never rewrite the whole file, and keep the other
agents intact.

## Fields

| Field | Type | Notes |
|---|---|---|
| `provider` | string | LLM provider, e.g. `anthropic`. |
| `model` | string | Model id, e.g. `claude-opus-5`. |
| `displayName` | string | Human-facing name. |
| `enabled` | boolean | Whether the gateway loads the agent. |
| `description` | string | One line shown in listings / `GET /api/agents`. |
| `isolationStrategy` | string | `in-process` or `subprocess`. |
| `thinking` | string | `none`, `low`, `medium`, or `high`. |
| `contextWindow` | number | Token budget, e.g. `200000`. |
| `toolIds` | string[] | Granted tools — subset of `read`, `write`, `edit`, `bash`, `grep`, `glob`, `todo`, `web_search`, `web_fetch`. Keep in sync with TOOLS.md. |
| `memory` | object | `{ "enabled": true, "indexing": "auto", "promptInjection": "full" }` |
| `soul` | object | `{ "enabled": true, "timezone": "UTC", "dayBoundary": "00:00", "reflectionOnSeal": false }` |
| `extensions` | object | Per-extension config; commonly `{ "botnexus-skills": { "enabled": true, "maxLoadedSkills": 20, "allowSkillCreation": false, "allowSkillDeletion": false } }` |

## Worked example

```json
{
  "agents": {
    "research-assistant": {
      "provider": "anthropic",
      "model": "claude-opus-5",
      "displayName": "Research Assistant",
      "enabled": true,
      "description": "Conducts research and synthesizes findings.",
      "isolationStrategy": "in-process",
      "thinking": "medium",
      "contextWindow": 200000,
      "toolIds": ["read", "write", "edit", "grep", "glob", "todo"],
      "memory": { "enabled": true, "indexing": "auto", "promptInjection": "full" },
      "soul": { "enabled": true, "timezone": "UTC", "dayBoundary": "00:00", "reflectionOnSeal": false },
      "extensions": {
        "botnexus-skills": {
          "enabled": true,
          "maxLoadedSkills": 20,
          "allowSkillCreation": false,
          "allowSkillDeletion": false
        }
      }
    }
  }
}
```

## Steps

1. Choose the `<id>` (kebab-case) — it must match `~/.botnexus/agents/<id>/`.
2. Add the agent object under `agents`, keyed by `<id>`, using the fields above.
3. Set `toolIds` to exactly the tools documented in the agent's TOOLS.md.
4. Save, restart the gateway, and confirm the agent shows in `GET /api/agents`.

## Pitfalls

- The **id is the object key** under `agents`, not a field inside the object.
- `thinking` and `isolationStrategy` accept only the enumerated values above.
- `toolIds` must be real tool ids; an unknown id is not granted.
- `config.json` holds secrets — make a targeted edit, don't regenerate the file.
- Restart is required for the gateway to pick up a new or changed agent.
