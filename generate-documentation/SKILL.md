---
name: generate-documentation
description: >-
  Generate structured documentation for codebases by analyzing repository
  structure, code patterns, and architecture. Produces a navigable docs/
  directory covering architecture, APIs, patterns, setup, and dependencies.
  Activate when users ask to document a repository, generate docs, create
  project documentation, or explain a codebase. Works for individual repos
  and multi-repo workspaces across any language or framework.
---

# Documentation Generator

Analyze a codebase and produce structured, accurate documentation. The output is a documentation directory with an index and topic pages that help developers and AI agents get productive quickly.

Leverage what already exists (README, doc comments, existing docs), then fill gaps with what the code reveals but nobody wrote down. Don't rewrite what's already documented — reference it, summarize it, and build on it.

## Workflow

### Step 1: Map the territory

Goal: understand what kind of repo this is and what documentation already exists. Do both in parallel.

**A. Structure and stack:**
1. List the top-level directory tree (2 levels deep max)
2. Read package manifests and build config to identify language, framework, and build system
3. Count source files to gauge repo size: small (<30), medium (30-150), large (>150)

**B. Existing documentation:**
1. Find the documentation directory — check `docs/`, `documentation/`, `doc/`, or any folder containing multiple `.md` files that aren't source code
2. Read `README.md`, and inventory all markdown in the docs directory. For large docs directories, read index/overview files and sample representative docs rather than reading everything.
3. Assess what's already well-documented vs. what has gaps

The documentation directory you find becomes your output target. Generate new pages into the same location. If no documentation directory exists, create `docs/`.

**Multi-repo mode:** When the user specifies multiple repo paths, analyze each repo separately using this workflow, then produce a shared overview covering cross-repo dependencies, integration points, and a unified navigation index.

### Step 2: Identify key components

Selectively read high-signal files:

1. **Entry points** — main files, CLI commands, route definitions, exported module indexes
2. **Public API surface** — exported functions, types, interfaces, REST endpoints
3. **Core modules** — the 3-5 directories where the real logic lives
4. **Configuration** — environment variables, feature flags, config schemas

Prioritization:
- Entry points and public interfaces first
- Core module index files (not every file in a module)
- Skip: tests, generated code, vendor/node_modules, lock files, assets
- Large repos: sample representative files from each major directory
- Monorepos: detect workspace/package boundaries; document packages and their relationships

### Step 3: Generate documentation

Produce documentation in the directory identified in Step 1. When existing docs are present, preserve their structure and add missing pages where they fit. When creating from scratch, use this structure:

```
{docs_dir}/
├── index.md           # Entry point: what, why, who, navigation
├── architecture.md    # Components, relationships, data flow
├── getting-started.md # Prerequisites, install, first run
├── api/               # Public interfaces (when multiple surfaces exist)
│   ├── index.md
│   └── {topic}.md
├── patterns.md        # Non-obvious conventions and design decisions
├── dependencies.md    # Key external deps and internal module map
└── development.md     # Build, test, contribute
```

Use subdirectories when a topic has multiple sub-areas. No page should exceed 300 lines — split into a subdirectory with an index if needed.

**Page rules:**

**index.md** — What is this, what problem does it solve, who is it for. Technology summary table and navigation links to all other pages. Derive from README + code structure, don't copy the README.

**architecture.md** — Key directories and their purpose. How components relate (data flow, control flow). Entry points and execution paths. Include a text diagram if the architecture has clear layers or boundaries.

**getting-started.md** — Prerequisites, installation, configuration, first run. If the README already covers this well, keep it short and reference the README.

**api/** — REST endpoints, CLI commands, exported functions/types. Split by resource or domain. Only include if the repo has a meaningful public interface that isn't already well-documented.

**patterns.md** — The highest-value page. Document 3-7 non-obvious patterns: error handling conventions, naming rules, architectural decisions visible in code but not written down. Focus on what would surprise a new developer or trip up an AI agent.

**dependencies.md** — Key external dependencies and why they're used (not a full list — the ones that shape the architecture). Internal module dependency map for repos with multiple packages.

**development.md** — How to build, run tests, lint, deploy. Include `mkdocs serve` for docs preview and link to https://www.mkdocs.org/user-guide/installation/ if needed.

Omit any page that doesn't apply to the repo. Don't generate empty sections.

### Step 4: Generate site configuration

Create `mkdocs.yml` in the repo root with Material theme, `docs_dir` set to the discovered documentation directory, and nav entries for all generated and existing pages. If `mkdocs.yml` already exists, update its nav to include new pages rather than overwriting.

### Step 5: Quality checks

Before finalizing:

- Every documented function, endpoint, or component actually exists in the codebase
- No hallucinated file paths or invented APIs
- Cross-references use relative links that resolve within the docs directory
- Links to repo-root files (like CONTRIBUTING.md) use the GitHub URL derived from git remote, not relative paths outside the docs root
- mkdocs.yml nav includes all markdown files in the docs directory and subdirectories

## Constraints

- Document at module level. Public interfaces and module boundaries, not internal implementation. Most modules get a paragraph; go deeper only for architecturally significant modules.
- Directory listings before file reads. Config files before source files. Stop reading when you have enough to document accurately.
- Don't invent information — if you can't determine something from the code, omit it.
- Don't duplicate the README — reference it, build on it, fill its gaps.
- Don't document test files, generated code, or vendored dependencies.
