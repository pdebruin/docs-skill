# generate-documentation

A GitHub Copilot skill that produces structured, navigable documentation for codebases. Analyzes repo structure, code patterns, and architecture to generate a browsable docs site.

## How it works

The skill follows a 5-step workflow:

1. Map the territory — directory tree, config files, existing docs (cheap context first)
2. Identify key components — entry points, public APIs, core modules
3. Generate documentation — index, architecture, patterns, dependencies, development
4. Generate site configuration — mkdocs.yml for browsable output
5. Quality checks — verify cross-references, file paths, line limits

Output is a `docs/` directory (or the repo's existing docs directory) with an `mkdocs.yml` at the repo root.

## Key design decisions

- Single skill handles both individual repos and multi-repo workspaces
- Pure prompt — no bundled scripts or MCP servers
- Always produces directory structure (not single-file output)
- Generates into existing docs directory if one exists; creates `docs/` otherwise
- Module-level documentation, not class/function level
- mkdocs-material for the browsable site (widely available, good defaults)

## Approach to existing documentation

The skill operates in two modes depending on what it finds:

- No existing docs — generates a full documentation set from scratch
- Existing docs directory — reads everything, assesses coverage, generates only gap-filling pages into the same directory without modifying existing files

This is a deliberate choice. When existing docs are present, the question of what to do with overlapping content (overwrite? rename? append? merge?) is a matter of opinion that varies by team and project. The skill avoids that judgment call entirely: it respects what's there, fills what's missing, and leaves conflict resolution to the human.

## Test results

Tested against 5 diverse repos:

| Repo | Type | Existing docs | Lines generated | AI Units | Behavior |
| --- | --- | --- | --- | --- | --- |
| xrm | .NET Blazor app | None | 696 | 137 | Full generation |
| afterburner | Go CLI | 339-line Architecture.md + 10 other docs | 298 (3 gap-fill pages) | ~125 | Gap-fill only |
| afterburner-fuel | Skills library | None | 764 | 73.6 | Full generation, adapted to config-heavy repo |
| afterburner-marketplace | Plugin marketplace | None | 404 | 68.9 | Full generation, plugins/ subdirectory |
| afterburner (no docs) | Go CLI | None | 130-line architecture.md | ~125 | Full generation |

Key observations:
- Token usage scales with repo complexity, not just repo size
- Small config-heavy repos (marketplace, fuel) get concise docs cheaply
- The skill adapts output structure to repo type (api/ for apps, plugins/ for marketplaces)
- Gap-fill mode correctly identifies what's missing without duplicating existing content

## Architecture comparison: generated vs human-written

The afterburner repo has a human-written 339-line Architecture.md (encyclopedic, function-level detail). When the skill generates architecture from scratch (afterburner2 test), it produces a 130-line module-level orientation doc with:

- 3-tier layer diagram (cmd / orchestration / infrastructure)
- Key abstractions with interface snippets
- Compact numbered data flow walkthroughs
- Configuration hierarchy
- Output modes table

The human version documents every function in every package. The skill version prioritizes "understand the architecture in 2 minutes." Different purposes — the skill produces orientation docs, not API reference. This is deliberate (skill instructions say "document at module level, not class level").

When both exist in afterburner4, the generated patterns.md and dependencies.md complement the detailed Architecture.md well. But the high-level framing (layer diagram, key abstractions summary) is absent because the skill saw the file and backed off entirely.

## Usage

Place the skill in `.github/skills/generate-documentation/SKILL.md` in any repo, then:

```
copilot "please generate documentation for this repo"
```

To browse the generated docs locally:

```
mkdocs serve
```

To publish to GitHub Pages:

```
mkdocs gh-deploy
```

If mkdocs is not installed, see https://www.mkdocs.org/user-guide/installation/

## File structure

```
generate-documentation/
└── SKILL.md          # The skill (submission artifact)
requirements.md       # Formal requirements with design decisions
input.md              # Raw challenge inputs (rules, rubric, tips)
README.md             # This file
```

## Scoring alignment

The skill targets all 7 rubric criteria (10 pts each, 70 max):

- Output Quality — module-level docs with patterns page as high-value differentiator
- Token Efficiency — cheap context first (directory listings before file reads), scales down for small repos
- Coverage — architecture, API, patterns, dependencies, development, getting-started
- Usability — browsable mkdocs site, clear navigation, cross-referenced pages
- Creativity — patterns.md documents what code reveals but nobody wrote down; adaptive structure per repo type
- Repo Compatibility — tested across Go CLI, .NET Blazor, skills library, plugin marketplace
- Fun Factor — TBD
