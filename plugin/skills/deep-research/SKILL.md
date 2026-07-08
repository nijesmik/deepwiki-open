---
name: deep-research
description: Run DeepWiki's multi-iteration Deep Research process on a specific topic in the current repository - research plan, iterative investigation updates, and a final conclusion, each grounded in fresh codebase exploration. Usage - /deepwiki:deep-research <topic or question>
---

# DeepWiki: Deep Research

Investigate a specific topic in the repository in the current working directory using DeepWiki's original Deep Research process. The three iteration prompts are preserved verbatim in `${CLAUDE_PLUGIN_ROOT}/templates/` — do not paraphrase or alter them.

The research topic is: `$ARGUMENTS`

## Placeholders

Fill these the same way in every template:
- `{repo_type}` — `github`, `gitlab`, or `bitbucket` based on the `origin` remote host, or `local` if there is no remote
- `{repo_url}` — the `origin` remote URL (or the local path)
- `{repo_name}` — the repository name
- `{language_name}` — the language of the user's request
- `{research_iteration}` — the current iteration number (intermediate template only)

## Process

Run up to 5 research iterations in sequence within this one invocation, mirroring the original frontend's automatic "continue the research" loop:

1. **Iteration 1** — Read and fill `${CLAUDE_PLUGIN_ROOT}/templates/deep-research-first.md`. Explore the repository (Grep/Glob/Read) to locate the code relevant to the topic, then produce the response it specifies (starting with `## Research Plan`, ending with `## Next Steps`).
2. **Iterations 2–4** — For each, read and fill `${CLAUDE_PLUGIN_ROOT}/templates/deep-research-intermediate.md` with the current iteration number. Before writing, do fresh exploration targeted at the gaps identified in the previous iteration's Next Steps. Produce the response it specifies (starting with `## Research Update {research_iteration}`). If the topic is already exhaustively covered, you may skip ahead to the final iteration.
3. **Final iteration (no later than 5)** — Read and fill `${CLAUDE_PLUGIN_ROOT}/templates/deep-research-final.md` and produce the comprehensive conclusion it specifies (starting with `## Final Conclusion`).

Output all iterations in order as one continuous response, each with its required heading. Every claim must be grounded in code you actually read during that iteration's exploration, with file and line citations.
