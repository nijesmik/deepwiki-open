# DeepWiki — Claude Code Plugin

[DeepWiki](https://github.com/AsyncFuncAI/deepwiki-open) repackaged as a Claude Code plugin. The original Next.js frontend, FastAPI backend, and RAG layer (embeddings, FAISS retrieval, multi-provider LLM clients) are replaced by Claude Code itself: the prompts stay, the plumbing goes.

- **Prompts are verbatim.** DeepWiki's prompts are preserved character-for-character in [`templates/`](templates/), with one documented exception (deep-research, see the provenance table). Only the runtime substitutions the original code performed (repo name, page title, language, file lists) remain as placeholders.
- **RAG → agentic exploration.** Where the backend injected embedding-retrieved file chunks into `<START_OF_CONTEXT>`, Claude Code agents now Read/Grep/Glob the repository directly.
- **Web renderer → plain files.** The original web app's renderer resolved source links, `Sources:` citations, and cross-page links; since output is now Markdown files in `.deepwiki/`, the page-writer agent fills those links with working relative paths (`../../file#L10-L20`, sibling `<page-id>.md`) while keeping the visible citation text the prompts specify.

## Install

```
/plugin marketplace add nijesmik/plugins
/plugin install deepwiki@nijesmik
```

The plugin is distributed through the [nijesmik/plugins](https://github.com/nijesmik/plugins) marketplace. Only this `plugin/` directory is installed — the rest of the repository (the original web app) does not come with it.

## Usage

Run inside the repository you want to document (clone it first if it's remote):

| Command | What it does |
|---|---|
| `/deepwiki:wiki [comprehensive\|concise] [lang]` | Generates a full wiki into `.deepwiki/` — structure XML, one Markdown page per topic (Mermaid diagrams, `Sources:` citations), and an index. Pages are written by parallel `page-writer` agents. |
| `/deepwiki:update [base-ref] [restructure]` | Updates an existing `.deepwiki/` wiki: diffs the repository against the wiki's generation baseline and regenerates only the pages whose source files changed. `restructure` re-runs structure determination (recommended after large changes). |
| `/deepwiki:ask <question>` | Answers a question about the codebase in DeepWiki's Ask style: direct, grounded, cited. |
| `/deepwiki:deep-research <topic>` | Deep Research on one topic: an exhaustive codebase investigation followed by a comprehensive, code-cited `## Final Conclusion`. |

Supported language codes (same as the original): `en`, `ja`, `zh`, `zh-tw`, `es`, `kr`, `vi`, `pt-br`, `fr`, `ru`.

## Updating a wiki

The original web app's only update path was the **Refresh Wiki** button: `DELETE /api/wiki_cache` followed by full regeneration. Because this plugin's output is plain files instead of a cache blob, an incremental path becomes possible, and `/deepwiki:update` provides it entirely in the wrapper layer — it fills the same verbatim templates; no prompt text was added or changed:

- `/deepwiki:wiki` records `.deepwiki/metadata.json` (view, language, the commit it generated from, and a dirty-worktree flag). `/deepwiki:update` diffs the repository against that baseline (an explicit ref argument overrides it; the fallback is the last commit touching `.deepwiki/`), maps changed files to pages via each page's `<relevant_files>` in `structure.xml`, and regenerates only the stale pages with the same page-writer agents.
- Changed files no page covers are assigned to the closest page; when changes outgrow the existing structure (new subsystems, most pages stale), the skill recommends `restructure`, which re-runs structure determination and regenerates around it while carrying over untouched pages.
- Committing `.deepwiki/` to git is recommended: updates then show up as reviewable diffs, and the wiki's own history doubles as a fallback baseline.

Equivalent of the original full refresh: re-run `/deepwiki:wiki`, which overwrites `.deepwiki/` from scratch.

## Prompt provenance

| Template | Original source |
|---|---|
| `templates/wiki-structure-comprehensive.md` / `-concise.md` | `src/app/[owner]/[repo]/page.tsx` (`determineWikiStructure`), comprehensive/concise branches of the same template literal |
| `templates/wiki-page.md` | `src/app/[owner]/[repo]/page.tsx` (`generatePageContent`) |
| `templates/ask.md` | `api/websocket_wiki.py` (simple chat system prompt) |
| `templates/deep-research.md` | **Adapted** (not verbatim) from `api/websocket_wiki.py`'s final-iteration Deep Research prompt: the original's five-iteration chat loop existed because each stateless API call saw only RAG snippets, so the multi-turn bookkeeping ("review the conversation history", "previous iterations", "Continue the research") was removed and the skill runs one exhaustive investigation instead. Guidelines are otherwise unchanged. |

Placeholder conventions follow each source: `${...}` for prompts that were JS template literals, `{...}` for prompts that were Python f-strings.

## What was intentionally dropped

Web UI and Mermaid renderer (Markdown output renders on GitHub/editors), embedding pipeline and wiki cache server (`api/data_pipeline.py`, `api/rag.py`, cache endpoints), multi-provider model configuration, and access-token handling — all of these are concerns Claude Code already covers.
