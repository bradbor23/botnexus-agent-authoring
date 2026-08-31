---
name: authoring-botnexus-agent-files
description: Use when creating or editing a BotNexus agent's markdown files — AGENTS.md, SOUL.md, TOOLS.md, BOOTSTRAP.md, IDENTITY.md, USER.md. Explains each file's sections, what belongs where, the on-disk layout under ~/.botnexus/agents/<id>/, and the order the gateway loads them in. Also covers why none of them are required, and why WORLD.md is a world-level file at ~/.botnexus/WORLD.md rather than an agent file. Pair with the botnexus-agent-config-entry skill to also register the agent in config.json.
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
  AGENTS.md      # peer agents, coordination patterns, memory notes
  SOUL.md        # personality, values, communication style, boundaries
  TOOLS.md       # per-tool usage guidance
  BOOTSTRAP.md   # first-run setup instructions
  IDENTITY.md    # name, role, expertise, how to address
  USER.md        # preferences the agent should respect
  MEMORY.md      # the agent's own memory (governed by memory config, not written by hand)
```

**None of these files is required.** An agent with none of them runs fine — the gateway
loads whichever exist and skips the rest. Write the ones that carry real content; an absent
file is better than a hollow one.

That list **is the load order**, and the order matters: earlier files are read first.
It comes from `WorkspaceContextBuilder.DefaultPromptFiles` and applies whenever the agent's
`systemPromptFiles` is empty. Setting `systemPromptFiles` in `config.json` replaces the
default list entirely — including files not named here.

> **WORLD.md is NOT an agent file.** It is read from **`~/.botnexus/WORLD.md`** — one
> world-level file shared by every agent — and injected ahead of everything else. A
> `WORLD.md` placed inside `~/.botnexus/agents/<id>/` is silently ignored unless you name it
> explicitly in `systemPromptFiles`. Put environment-wide rules in the world file; put
> agent-specific context in that agent's own files.

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

## USER.md and BOOTSTRAP.md — the rest of the load order

`USER.md` (`# User`) records preferences the agent should respect. `BOOTSTRAP.md` carries
first-run setup instructions. Both are in the default load order, so both are read when
present. Create them only if you have content — omit the file entirely rather than leaving
an empty one.

`MEMORY.md` is also in the load order but is the agent's own; it is governed by the
`memory` config and should not be hand-authored.

For environment-wide rules, edit **`~/.botnexus/WORLD.md`** — the shared world file — rather
than adding a `WORLD.md` to the agent directory, which is not read.

## Steps

1. Pick a kebab-case `<id>` and create `~/.botnexus/agents/<id>/`.
2. Write SOUL.md and IDENTITY.md first — they anchor the agent's behaviour.
3. Write AGENTS.md and TOOLS.md; make sure the `### <tool>` blocks match the tools you
   intend to grant.
4. Add USER.md / BOOTSTRAP.md only if you have real content for them. For world-wide rules
   edit `~/.botnexus/WORLD.md` instead — a per-agent WORLD.md is not loaded.
5. Register the agent in `config.json` — use the `botnexus-agent-config-entry` skill.
6. Restart the gateway (or reload) and confirm the agent appears in `GET /api/agents`.

## Pitfalls

- **Filenames are uppercase** (`SOUL.md`, not `soul.md`) and the directory `<id>` must
  match the `config.json` key exactly.
- Keep the section headings above — they are the structure the files are read against.
- Don't invent facts in SOUL/IDENTITY; describe intended behaviour, not achievements.
- An empty file is worse than no file — skip any you have nothing real to put in.
- **A `WORLD.md` in the agent directory does nothing.** The world file the gateway reads is
  `~/.botnexus/WORLD.md`, and it applies to every agent.
- These files are optional, not required. If an agent misbehaves, missing markdown is rarely
  the cause — check `toolIds` and the `config.json` entry first.
