# DeepWiki — Claude Code Plugin

A fork of [AsyncFuncAI/deepwiki-open](https://github.com/AsyncFuncAI/deepwiki-open) repackaged as a **Claude Code plugin**. DeepWiki automatically creates wikis for a code repository — analyzing the code structure, generating comprehensive documentation with Mermaid diagrams and source citations, and organizing it into navigable pages.

This fork keeps DeepWiki's prompts and drops its infrastructure: the Next.js frontend, FastAPI backend, and RAG layer (embeddings, FAISS retrieval, multi-provider LLM clients) are replaced by Claude Code itself, which explores the repository directly with its own tools. See [`plugin/README.md`](plugin/README.md) for the full design, prompt provenance, and what was intentionally changed.

## Install

```
/plugin marketplace add nijesmik/plugins
/plugin install deepwiki@nijesmik
```

This plugin is distributed through the [nijesmik/plugins](https://github.com/nijesmik/plugins) marketplace, and only the [`plugin/`](plugin/) directory of this repository is installed.

## Usage

Run inside the repository you want to document (clone it first if it's remote):

| Command | What it does |
|---|---|
| `/deepwiki:wiki [comprehensive\|concise] [lang]` | Generates a full wiki into `.deepwiki/` — structure, one Markdown page per topic, and an index. |
| `/deepwiki:ask <question>` | Answers a question about the codebase: direct, grounded, cited. |
| `/deepwiki:deep-research <topic>` | Exhaustive investigation of one topic, ending in a comprehensive, code-cited conclusion. |

## Repository layout

- [`plugin/`](plugin/) — the Claude Code plugin (the only part that gets installed; the marketplace manifest lives in [nijesmik/plugins](https://github.com/nijesmik/plugins))
- `src/`, `api/` — the original DeepWiki web application, kept unmodified as the provenance for the plugin's verbatim prompt templates; for running it as a web app, use the [original project](https://github.com/AsyncFuncAI/deepwiki-open)

## Credits & License

All prompts and the wiki-generation design come from [DeepWiki-Open](https://github.com/AsyncFuncAI/deepwiki-open) by AsyncFuncAI. Licensed under the MIT License — see [LICENSE](LICENSE).
