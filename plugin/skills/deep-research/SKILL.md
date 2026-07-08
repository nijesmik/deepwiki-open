---
name: deep-research
description: Run DeepWiki-style Deep Research on a specific topic in the current repository - one exhaustive codebase investigation followed by a comprehensive, code-cited conclusion. Usage - /deepwiki:deep-research <topic or question>
disable-model-invocation: true
---

# DeepWiki: Deep Research

Investigate a specific topic in the repository in the current working directory, DeepWiki Deep Research style. The original DeepWiki ran this as five prompted chat iterations because each stateless API call could only see RAG-retrieved snippets; in Claude Code you explore the repository directly, so this skill runs a single exhaustive investigation followed by one synthesis. The synthesis prompt in `${CLAUDE_PLUGIN_ROOT}/templates/deep-research.md` is adapted from the original final-iteration prompt (multi-turn bookkeeping removed, guidelines otherwise unchanged) — do not paraphrase or alter it further.

The research topic is: `$ARGUMENTS`

## Phase 1 — Exhaustive investigation

Explore the repository with Grep, Glob, and Read until new searches stop yielding new information about the topic. Stay strictly on the topic; do not survey the repository in general. Cover at minimum:

- Where the topic is defined or implemented (primary files, key functions/classes)
- Where it is used: callers, imports, references across the codebase
- How it is configured: config files, environment variables, constants, defaults
- Its lifecycle and data flow: inputs, transformations, outputs, error paths
- Related tests and documentation, if any exist
- Edge cases, TODOs, or inconsistencies visible in the code

Read the actual code at each site — do not conclude anything from file names or search hits alone. Keep track of file paths and line numbers for every finding; they are required as citations in the synthesis.

## Phase 2 — Synthesis

1. Read `${CLAUDE_PLUGIN_ROOT}/templates/deep-research.md` and fill its placeholders:
   - `{repo_type}` — `github`, `gitlab`, or `bitbucket` based on the `origin` remote host, or `local` if there is no remote
   - `{repo_url}` — the `origin` remote URL (or the local path)
   - `{repo_name}` — the repository name
   - `{language_name}` — the language of the user's request
2. Produce the response it specifies: one comprehensive conclusion starting with `## Final Conclusion`, strictly focused on the topic, with every claim grounded in the code you read during Phase 1 and cited with file paths and line numbers.
