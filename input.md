# Generate Documentation Skill — Requirements

## Challenge Context

Challenge 001 from Afterburner. Deadline: May 25, 2026. Judging includes private evaluation repos to test generalizability.

## What We Are Building

A skill (SKILL.md with valid YAML frontmatter) that produces useful, accurate documentation for codebases. Must work with GitHub Copilot (CLI or VS Code).

## Categories

- Individual Repo: documentation for a single repository
- Multi-Repo: documentation spanning multiple repositories

## Output Expectations

Documentation should cover:
- Architecture
- APIs
- Patterns and conventions
- Dependencies
- Setup instructions
- Anything a developer or AI needs to get productive quickly

Must adapt to different repo structures (Go CLI, Astro site, Java project, etc.).

## Scoring Criteria (7 criteria, 10 pts each, 70 max)

| Criterion | Description |
|-----------|-------------|
| Output Quality | Accuracy, correctness, usefulness of generated documentation |
| Token Efficiency | How well the skill manages token budget; clever context compression |
| Coverage | Completeness — architecture, APIs, patterns, dependencies, cross-repo relationships |
| Usability | Ease of setup and use; clear instructions; minimal configuration |
| Creativity | Novel approaches, clever prompting strategies, innovative techniques |
| Repo Compatibility | Works across diverse repos (different languages, sizes, structures) |
| Fun Factor | Entertaining, clever, or delightful — subjective bonus, won't penalize normal submissions |

Judging: Juan (primary) + volunteers. Community vote (PR reactions) as tiebreaker. Private evaluation repos for generalizability.

## Submission Structure

```
submissions/001-generate-documentation/{category}/{alias}/
├── SKILL.md       # Required — valid YAML frontmatter (name, description)
└── README.md      # Required — approach, usage instructions, example output
```

Categories: `individual/` or `multi-repo/`. Separate PR per category.
PR naming: `submission/001/{category}/{alias}`

Optional files: scripts, MCP configs, supporting assets, sample output.

PR stays open during challenge period. Community votes via reactions. Winning PRs merged after judging.

## Test Repos

- afterburner — CLI tool (Go)
- afterburner-marketplace — Plugin marketplace
- afterburner-pages — Docs site (Astro + Starlight)
- afterburner-fuel — Curated skill library

Judging will also use private repos we cannot see in advance.

## Design Constraints

- Token efficiency is explicitly scored — cannot just dump entire repos into context
- Must generalize across repo types and languages
- Skill must be invocable as a Copilot skill (not a standalone tool)

## Agent Skills Spec (from agentskills.io)

Format constraints:
- name: max 64 chars, lowercase + hyphens, must match parent directory name
- description: max 1024 chars, should describe what + when
- SKILL.md body: recommended < 500 lines, < 5000 tokens
- Progressive disclosure: metadata loaded at startup, body on activation, references on demand
- Optional dirs: scripts/, references/, assets/

Best practices (key points for scoring well):
- Ground in real expertise, not generic LLM knowledge
- Add what the agent lacks, omit what it knows (don't explain HTTP, do explain project-specific conventions)
- Provide defaults, not menus — pick one approach, mention alternatives briefly
- Favor procedures over declarations — teach how to approach, not what to produce
- Include gotchas sections for non-obvious facts
- Use templates for output format (agents pattern-match against concrete structures)
- Use checklists for multi-step workflows
- Use validation loops (do work → validate → fix → repeat)
- Structure large skills with progressive disclosure (core in SKILL.md, detail in references/)
- Aim for moderate detail — concise stepwise guidance + working example > exhaustive docs
- Calibrate control: be prescriptive where fragile, give freedom where multiple approaches are valid

Token budget strategy:
- SKILL.md loads fully on activation — every token competes for attention
- Move detailed reference material to separate files, tell agent when to load each
- Keep instructions focused on the current task class

Frontmatter:
```yaml
---
name: generate-documentation
description: "..."
---
```

Body is a system prompt that instructs the model. The starter template uses a 4-step workflow:
1. Analyze Repository Structure (directory tree, config files, existing docs, entry points)
2. Identify Key Components (source dirs, patterns, API surfaces, dependencies)
3. Generate Documentation (overview, architecture, getting started, API ref, patterns, deps, dev guide)
4. Output (single DOCS.md for small repos, directory structure for large repos)

Token-saving strategies suggested in template:
- Directory listings before file reads
- Read config files first (cheap architecture signal)
- Summarize into intermediate notes between passes (compress context)
- Multi-pass: structure first, then details on important components
- Sampling: representative files from each directory rather than everything

Approach ideas from template:
- File-type prioritization: config → entry points → core modules → utilities → tests
- Monorepo strategy: document modules independently, then cross-module overview
- Small library strategy: focus on public API surface and usage examples
- Multi-repo extension: shared vocabulary, service boundaries, communication patterns

## Open Questions

- What are the scoring rubric details? Page linked but not yet fetched.
- How do we differentiate from the starter template? (it is a baseline, not a winning entry)
- Single skill or a set of skills? Challenge allows either.
- How do we handle multi-repo category differently from individual?
- What output format wins on quality score? (structured files vs single doc)

## Tips (from challenge page)

- Be specific — vague instructions produce vague output. Tell the model exactly what to look for and how to structure the output.
- Manage tokens — large repos can exhaust the context window. Strategies: prioritize key files, summarize before deep-diving, multi-pass approaches.
- Test on diverse repos — a skill that only works on one language/structure won't score well on Repo Compatibility.
- Include example output — README.md must show what the skill produces for at least one real repo.

## References

- llm-wiki — AI-generated wiki pages from codebases
- DeepWiki — Deep documentation generation
- graphify — Graph-based repo understanding
- psh-docs — Internal documentation generation exploration
- /home/pieterd/_projects/azarch-skill/hybrid-azarch/ — reference skill demonstrating hub + references pattern, progressive disclosure, decision tree routing
