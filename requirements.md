# Generate Documentation Skill — Requirements

## Purpose

An agent skill that generates useful, accurate documentation for codebases. It analyzes repository structure, code patterns, and architecture to produce documentation that helps developers and AI agents get productive quickly.

The skill must work with GitHub Copilot (CLI or VS Code) and adapt to diverse repo structures (Go CLIs, Astro sites, Java projects, Python libraries, monorepos, etc.).

## Audience

- Developers onboarding to an unfamiliar codebase
- AI agents needing context to produce accurate code suggestions
- Teams maintaining documentation alongside code

## Categories

- **Individual Repo** — Generate documentation for a single repository
- **Multi-Repo** — Generate documentation spanning multiple related repositories

## Behavioral Requirements

### BR-1: Structure Before Content

The skill must establish understanding of the repository's shape before reading source files. Directory trees and config files are cheap context; source files are expensive. The agent should map the full structure before selectively reading files.

### BR-2: Adaptive Depth

The skill must adapt its approach to the repo it encounters. A 10-file Go CLI needs a different treatment than a 500-file monorepo. The agent should:

- Detect repo type, size, and language from structural signals
- Adjust documentation scope and detail accordingly
- Not apply a one-size-fits-all template rigidly

### BR-3: Token Discipline

The skill must manage context window budget deliberately:

- Prioritize high-signal files (entry points, configs, public APIs) over low-signal files (tests, generated code, vendor)
- Summarize intermediate findings between passes rather than re-reading
- Use sampling for large directories (representative files, not exhaustive reads)
- Never attempt to read an entire large repo into context

### BR-4: Accuracy Over Speculation

The skill must produce documentation grounded in what the code actually does:

- State facts derived from reading the code, not assumptions about what it might do
- When uncertain, say so rather than hallucinate
- Distinguish between documented behavior and inferred behavior

## Functional Requirements

### FR-1: Project Overview

Generate a clear summary of what the project is, what problem it solves, and who it is for. Derive this from README, package metadata, entry points, and code structure — not generic boilerplate.

### FR-2: Architecture Documentation

Identify and document the high-level structure:

- Key directories and their purpose
- How components relate to each other
- Data flow and control flow patterns
- Entry points and execution paths

### FR-3: Getting Started

Produce setup and usage instructions:

- Prerequisites and dependencies
- Installation steps
- Basic usage / running the project
- Environment configuration

### FR-4: API Surface

Document the public interface:

- Exported functions, types, and interfaces
- REST/GraphQL endpoints (if applicable)
- CLI commands and flags (if applicable)
- Configuration options

### FR-5: Patterns and Conventions

Identify and document coding patterns used in the repo:

- Naming conventions
- Error handling patterns
- Testing patterns
- Architecture decisions visible in code structure

### FR-6: Dependency Map

Document key external dependencies:

- What they are and why they are used
- Version constraints
- Internal cross-references between packages/modules

### FR-7: Multi-Repo Relationships (Multi-Repo category only)

When documenting multiple repos:

- Identify shared vocabulary and concepts across repos
- Map service boundaries and communication patterns
- Document cross-repo dependencies and integration points
- Produce a unified overview that ties repos together

## Non-Functional Requirements

### NFR-1: Repo Compatibility

The skill must work across diverse repositories without modification:

- Multiple languages (Go, TypeScript, Python, Java, Rust, etc.)
- Multiple project types (CLI, web app, library, API service, docs site)
- Multiple sizes (small single-purpose libs to large monorepos)
- Multiple structures (flat, nested, monorepo with workspaces)

### NFR-2: Token Efficiency

The skill must produce high-quality output without burning through token budgets. Strategies should be explicit in the skill instructions and measurable in practice.

### NFR-3: Output Quality

Generated documentation must be:

- Accurate (matches what the code does)
- Useful (helps someone get productive)
- Well-structured (clear hierarchy, scannable)
- Appropriately scoped (not too shallow, not exhaustively detailed)

### NFR-4: Usability

The skill must be easy to use:

- Minimal or no configuration required
- Clear invocation (single command or prompt)
- Output is immediately useful without post-processing

### NFR-5: Determinism

Given the same repo, the skill should produce reasonably consistent output structure across runs. The content may vary with model behavior, but the approach and coverage areas should be stable.

## Design Decisions

1. **Single skill for both categories.** One skill handles individual and multi-repo. Multi-repo adds a cross-repo synthesis step as a conditional branch, not a separate skill. Submit to both categories.
2. **Always directory structure output.** Consistent output format regardless of repo size. Smaller repos get shorter files within the same structure. Predictable and templatable.
3. **Pure prompt, no scripts.** The agent has bash/glob/grep/view tools already. No bundled scripts — fewer dependencies, better usability, less maintenance surface.
4. **Hub + references architecture (azarch pattern).** SKILL.md contains core workflow and decision tree routing. Reference files loaded on demand for specific situations (monorepo, multi-repo, language-specific heuristics).
5. **User provides multi-repo paths explicitly.** No auto-discovery. Unambiguous, predictable, avoids false positives.

## Output Structure

```
docs/
├── index.md           # Entry point: what this is, how to navigate, links to all pages
├── architecture.md    # Components, relationships, data flow
├── getting-started.md # Setup, prerequisites, first run
├── api.md             # Public interfaces (if applicable)
├── patterns.md        # Conventions, design decisions
├── dependencies.md    # External deps, internal module map
└── development.md     # Build, test, contribute
```

Pages are omitted if not applicable (e.g., a tiny CLI may skip api.md).
Cross-references between pages where natural (architecture links to relevant component details).

### Scaling Rules

1. **Document at module level, not class level.** Describe the forest, not every leaf. Public interfaces and module boundaries, not internal implementation details.
2. **Overflow threshold: 300 lines.** When a page exceeds 300 lines, split into a subdirectory with an index. E.g., `api.md` becomes `api/index.md` + `api/auth.md` + `api/orders.md`.
3. **Depth is earned.** Only go deeper into a module if it is the primary focus of the repo or is architecturally significant. Most modules get a paragraph, not a page.

## Workflow Observations (from xrm test run)

A manual documentation pass on the xrm project revealed this natural workflow:

1. **Inventory** — glob for all markdown + scan src/ tree. Understand what exists before creating anything.
2. **Deep read** — read existing docs to assess coverage, then read source to find gaps. The gap is usually API detail and setup instructions (scattered across README and ad-hoc docs).
3. **Structure** — map content to focus areas. Existing docs slot into the structure almost directly. Only genuinely new content comes from reading source that wasn't documented.
4. **Wiring** — create nav/index, move files, update cross-references, iterate until clean.

Key insight: treat existing documentation as the primary asset. The work is organization and gap-filling, not rewriting. New content comes from what the code reveals but nobody wrote down (API signatures from controllers, setup from Program.cs/Dockerfile, patterns from recurring code shapes).

Improvement: read source in parallel with markdown scan rather than sequentially — avoids a second pass for API/model details.

| Criterion | Addressed by |
|-----------|-------------|
| Output Quality | FR-1 through FR-6, NFR-3 |
| Token Efficiency | BR-3, NFR-2 |
| Coverage | FR-1 through FR-7 |
| Usability | NFR-4 |
| Creativity | Design decisions, novel approaches |
| Repo Compatibility | BR-2, NFR-1 |
| Fun Factor | TBD — personality, presentation, easter eggs |
