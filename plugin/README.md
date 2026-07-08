# DeepWiki — Claude Code Plugin

[DeepWiki](https://github.com/AsyncFuncAI/deepwiki-open) repackaged as a Claude Code plugin. The original Next.js frontend, FastAPI backend, and RAG layer (embeddings, FAISS retrieval, multi-provider LLM clients) are replaced by Claude Code itself: the prompts stay, the plumbing goes.

- **Prompts are verbatim.** DeepWiki's prompts are preserved character-for-character in [`templates/`](templates/), with one documented exception (deep-research, see the provenance table). Only the runtime substitutions the original code performed (repo name, page title, language, file lists) remain as placeholders.
- **RAG → agentic exploration.** Where the backend injected embedding-retrieved file chunks into `<START_OF_CONTEXT>`, Claude Code agents now Read/Grep/Glob the repository directly.

## Install

```
/plugin marketplace add nijesmik/deepwiki-open
/plugin install deepwiki@nijesmik
```

Only the `plugin/` directory is installed — the rest of the repository (the original web app) does not come with it.

## Usage

Run inside the repository you want to document (clone it first if it's remote):

| Command | What it does |
|---|---|
| `/deepwiki:wiki [comprehensive\|concise] [lang]` | Generates a full wiki into `.deepwiki/` — structure XML, one Markdown page per topic (Mermaid diagrams, `Sources:` citations), and an index. Pages are written by parallel `page-writer` agents. |
| `/deepwiki:ask <question>` | Answers a question about the codebase in DeepWiki's Ask style: direct, grounded, cited. |
| `/deepwiki:deep-research <topic>` | Deep Research on one topic: an exhaustive codebase investigation followed by a comprehensive, code-cited `## Final Conclusion`. |

Supported language codes (same as the original): `en`, `ja`, `zh`, `zh-tw`, `es`, `kr`, `vi`, `pt-br`, `fr`, `ru`.

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
