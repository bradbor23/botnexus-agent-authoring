# botnexus-agent-authoring

A **skills-only BotNexus plugin** — a small, self-contained set of skills that teach an
agent to hand-author BotNexus agents: the `SOUL/IDENTITY/AGENTS/TOOLS` markdown files and
the `config.json` agent entry.

It doubles as the **end-to-end pilot** for the BotNexus plugin marketplace: it exercises
the whole pipeline — fetch (`git clone`) → validate (`plugin.json` against the manifest
schema) → install (promote to `~/.botnexus/plugins/<name>/`) → discover (two skills) —
with **zero code**, so the pipeline can be proven before the code-carrying plugin format
lands. (It's the skills-shaped companion to the Agent Builder *extension*, which serves a
UI and therefore needs that future format.)

## Layout

```
.botnexus-plugin/plugin.json                    # the manifest (required, at repo root)
skills/
  authoring-botnexus-agent-files/SKILL.md       # the agent markdown files
  botnexus-agent-config-entry/SKILL.md          # the config.json agent entry
README.md
LICENSE
```

The whole repository is the plugin — one plugin per repo, at the root. On install
everything except `.git/` is copied verbatim into `~/.botnexus/plugins/<name>/`, so the
repo is deliberately lean (markdown only).

## Install

Install through the BotNexus marketplace by repository URL (it clones the repo, validates
the manifest, and stages the two skills). Skills are discovered by convention from
`skills/<name>/SKILL.md` — no `skills` array is declared in the manifest.

After install, the two skills are available to agents (note: plugin skills are lowest
priority in the skill merge, so a same-named global/agent/workspace skill wins — these are
namespaced `botnexus-*`/`authoring-*` to avoid being shadowed).

## Conformance to the plugin requirements

- `.botnexus-plugin/plugin.json` at the repo root; only allowed manifest fields.
- `name` is lowercase kebab-case and is authoritative for the install directory.
- Skills are exactly one level deep (`skills/<name>/SKILL.md`).
- Each `SKILL.md` opens with `---` frontmatter carrying a non-empty `description`
  (≤ 1024 chars); no skill `name` contains `--`.
- No build output or large assets — markdown only.

Tag a release commit to install as a pinned reference.
