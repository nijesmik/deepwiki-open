---
name: wiki
description: Generate a complete DeepWiki-style wiki for the current repository. Determines a wiki structure (sections/pages as XML), then writes each page with Mermaid diagrams and source citations via parallel page-writer agents. Output goes to .deepwiki/. Usage - /deepwiki:wiki [comprehensive|concise] [language-code]
disable-model-invocation: true
---

# DeepWiki: Generate Wiki

Generate a DeepWiki-style wiki for the repository in the current working directory. This skill reproduces the original DeepWiki pipeline (structure determination → per-page generation), with one difference: instead of a RAG retrieval backend supplying file contents, you and the page-writer agents explore the repository directly with Read/Grep/Glob.

The prompts used below are the original DeepWiki prompts, preserved verbatim in `${CLAUDE_PLUGIN_ROOT}/templates/`. Do not paraphrase or alter them — fill in their placeholders and follow them exactly.

## Arguments

`$ARGUMENTS` may contain, in any order:
- A view type: `comprehensive` (default) or `concise`
- A language code: one of `en` (default), `ja`, `zh`, `zh-tw`, `es`, `kr`, `vi`, `pt-br`, `fr`, `ru`

Resolve `${language_name}` from the language code using this mapping (identical to the original frontend):

| code | language_name |
|---|---|
| en | English |
| ja | Japanese (日本語) |
| zh | Mandarin Chinese (中文) |
| zh-tw | Traditional Chinese (繁體中文) |
| es | Spanish (Español) |
| kr | Korean (한국어) |
| vi | Vietnamese (Tiếng Việt) |
| pt-br | Brazilian Portuguese (Português Brasileiro) |
| fr | Français (French) |
| ru | Русский (Russian) |

## Step 1 — Gather repository inputs

This replaces the original frontend's GitHub API calls:

1. Determine `${owner}` and `${repo}` from `git remote get-url origin` (fall back to the working directory name for `${repo}` and leave `${owner}` as `local` if there is no remote).
2. Build `${fileTree}` by running `git ls-files` (one path per line). If the tree is extremely large (>10,000 files), you may summarize deeply nested vendored/generated directories, but keep all source directories complete.
3. Read the repository's `README.md` (or closest equivalent) as `${readme}`. If none exists, use an empty string.

## Step 2 — Determine the wiki structure

1. Read the structure prompt template:
   - comprehensive view: `${CLAUDE_PLUGIN_ROOT}/templates/wiki-structure-comprehensive.md`
   - concise view: `${CLAUDE_PLUGIN_ROOT}/templates/wiki-structure-concise.md`
2. Substitute `${owner}`, `${repo}`, `${fileTree}`, `${readme}`, and `${language_name}`.
3. Execute the filled prompt yourself and produce the `<wiki_structure>` XML it specifies. Base the page list and each page's `<relevant_files>` on the actual file tree — every `file_path` must exist in the repository.
4. Save the XML verbatim to `.deepwiki/structure.xml`.

## Step 3 — Generate each page

For every `<page>` in the structure XML:

1. Read `${CLAUDE_PLUGIN_ROOT}/templates/wiki-page.md` and substitute:
   - `${page.title}` — the page's `<title>` (all occurrences)
   - `${file_list}` — one line per `<file_path>`, formatted as `- [path/to/file](path/to/file)` (relative repository paths)
   - `${language_name}` — as resolved above
2. Launch a `deepwiki:page-writer` agent whose task message is:
   - the filled page prompt, followed by
   - `Output file: .deepwiki/pages/<page-id>.md` (using the page's XML `id`)
3. Run page agents in parallel (batches of 3–5, mirroring the original queue behavior). Wait for all pages to finish; re-run any page that fails once.

## Step 4 — Build the index

Write `.deepwiki/README.md` following the original Markdown export format:

```
# Wiki Documentation for <repo_url>

Generated on: <YYYY-MM-DD HH:MM:SS>

## Table of Contents

- [<page title>](pages/<page-id>.md)
...
```

- List pages grouped under their section titles when the comprehensive structure has `<sections>`.
- After the table of contents, add a short "Related Pages" mapping if pages declare `<related_pages>` (`Related topics: [Title](pages/id.md), ...`).

## Step 5 — Report

Tell the user: the wiki title and description, the list of generated pages with paths, and any pages that failed. Do not print full page contents in the conversation.
