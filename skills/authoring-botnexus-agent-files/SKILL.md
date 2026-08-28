---
name: authoring-botnexus-agent-files
description: Use when creating or editing a BotNexus agent's markdown files — SOUL.md, IDENTITY.md, AGENTS.md, TOOLS.md, and the optional WORLD.md / USER.md. Explains each file's sections, what belongs where, and the on-disk layout the gateway expects under ~/.botnexus/agents/<id>/. Pair with the botnexus-agent-config-entry skill to also register the agent in config.json.
license: MIT
compatibility: BotNexus gateway. Agent files live at ~/.botnexus/agents/<id>/ (uppercase filenames).
allowed-tools: read, write, edit
disable-model-invocation: false
metadata:
  category: agent-authoring
---

# Authoring BotNexus agent files

A BotNexus agent is a directory of uppercase markdown files under
`~/.botnexus/agents/<id>/`. `<id>` is the agent's stable, kebab-case id and must match
the key used in `config.json` (see the `botnexus-agent-config-entry` skill).

```
~/.botnexus/agents/<id>/
  SOUL.md        # personality, values, communication style, boundaries  (required)
  IDENTITY.md    # name, role, expertise, how to address                 (required)
  AGENTS.md      # peer agents, coordination patterns, memory notes       (required)
  TOOLS.md       # per-tool usage guidance                                (required)
  WORLD.md       # environment/context the agent operates in              (optional)
  USER.md        # preferences the agent should respect                   (optional)
```

Write each file as plain markdown with the section headings below. Keep statements
concrete and behavioural — these files are read as the agent's standing instructions.

## SOUL.md — who the agent is

```markdown
# Soul

## Personality
One short paragraph: temperament and disposition (e.g. "Calm, methodical, risk-aware;
biased toward automation and documenting decisions over heroics.").

## Core Values
- What it prioritises, one per line (reliability over novelty; least privilege; …)

## Communication Style
- How it should communicate (evidence-based; cites logs/metrics; concise; …)

## Boundaries
- Hard limits — what it must not do (no unreviewed prod changes during a freeze;
  escalates security incidents rather than containing alone; …)
```

## IDENTITY.md — its role

```markdown
# Identity

## Name
The agent's display name.

## Role
One or two sentences: primary responsibility and remit.

## Expertise
- Domains of expertise, one per line.

## How to address me
A short line on how the user should refer to it.
```

## AGENTS.md — how it coordinates

```markdown
# Agents

## This Gateway
<Display name> coordinates with:
- **Peer Agent Name** — what that peer is for

## Coordination Patterns
- How it delegates and consults (spawn sub-agents for verification; delegate
  specialised analysis; …)

## Memory Notes
- What to store, where, and for how long (store reusable methods as skills; track
  key context in conversation memory; …)
```

## TOOLS.md — how it uses its tools

```markdown
# Tools

## General Principles
- Cross-cutting rules (read before assuming; verify output before acting; never run
  destructive commands without confirmation; …)

## Tool-Specific Notes

### read
- Guidance for this tool.

### bash
- Guidance for this tool.
```

List one `### <tool>` block per tool the agent is granted. The tool ids match its
`toolIds` in `config.json` (`read`, `write`, `edit`, `bash`, `grep`, `glob`, `todo`,
`web_search`, `web_fetch`). Keep TOOLS.md and `toolIds` in sync.

## WORLD.md / USER.md — optional context

Create these only if you have content. `WORLD.md` (`# World`) describes the environment
the agent runs in; `USER.md` (`# User`) records user preferences it should respect. Omit
the file entirely rather than leaving an empty one.

## Steps

1. Pick a kebab-case `<id>` and create `~/.botnexus/agents/<id>/`.
2. Write SOUL.md and IDENTITY.md first — they anchor the agent's behaviour.
3. Write AGENTS.md and TOOLS.md; make sure the `### <tool>` blocks match the tools you
   intend to grant.
4. Add WORLD.md / USER.md only if you have real content for them.
5. Register the agent in `config.json` — use the `botnexus-agent-config-entry` skill.
6. Restart the gateway (or reload) and confirm the agent appears in `GET /api/agents`.

## Pitfalls

- **Filenames are uppercase** (`SOUL.md`, not `soul.md`) and the directory `<id>` must
  match the `config.json` key exactly.
- Keep the section headings above — they are the structure the files are read against.
- Don't invent facts in SOUL/IDENTITY; describe intended behaviour, not achievements.
- An empty optional file is worse than no file — skip WORLD.md/USER.md if unused.
