---
name: ask
description: Ask a question about the current repository, DeepWiki-style. Explores the codebase for evidence, then answers directly and concisely with file/line citations, in the language of the question. Usage - /deepwiki:ask <question>
argument-hint: "<question>"
---

# DeepWiki: Ask

Answer a question about the repository in the current working directory using DeepWiki's original Ask behavior. The system prompt is preserved verbatim in `${CLAUDE_PLUGIN_ROOT}/templates/ask.md` — do not paraphrase or alter it.

The question is: `$ARGUMENTS`

If no question was provided, ask the user what they want to know about the repository and stop.

## Steps

1. Read `${CLAUDE_PLUGIN_ROOT}/templates/ask.md` and fill its placeholders:
   - `{repo_type}` — `github`, `gitlab`, or `bitbucket` based on the `origin` remote host, or `local` if there is no remote
   - `{repo_url}` — the `origin` remote URL (or the local path)
   - `{repo_name}` — the repository name
   - `{language_name}` — the language of the user's question
2. Gather evidence. In the original, a RAG retriever supplied the top-matching code chunks; here, explore the repository directly with Grep, Glob, and Read until you have read the actual code relevant to the question. Do not answer from memory of the file tree alone.
3. Answer the question following the filled system prompt exactly — direct answer first, no preamble, markdown formatting within the answer, file paths and line numbers cited, strictly grounded in the code you read.
