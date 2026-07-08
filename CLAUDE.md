# CLAUDE.md

Fork of [AsyncFuncAI/deepwiki-open](https://github.com/AsyncFuncAI/deepwiki-open) repackaged as a Claude Code plugin. The deliverable is `plugin/`; everything else is frozen upstream code kept as provenance.

## The verbatim principle (core rule)

The prompt templates in `plugin/templates/` are **character-for-character copies** of the original DeepWiki prompts. This is the project's defining constraint — it keeps the templates mechanically diffable against upstream.

- **Never edit `plugin/templates/*.md`**, including fixing typos, awkward spacing (`IMPORTANT:You`), the hardcoded "GitHub repository" wording, or the `<!-- Add additional relevant files ... -->` comment. Original flaws are preserved on purpose.
- The only allowed contents besides original text are the runtime-substitution placeholders: `${...}` for prompts extracted from JS template literals, `{...}` for prompts extracted from Python f-strings. Do not normalize one style to the other.
- Known exception: `templates/deep-research.md` is a documented **adaptation** (multi-turn bookkeeping removed), not verbatim. Any new exception must be equally documented in the provenance table in `plugin/README.md`.

## Where changes go instead: the wrapper pattern

Behavior changes belong in the files we own — `plugin/skills/*/SKILL.md` and `plugin/agents/*.md` — never in templates:

- Defect caused by the **environment change** (web app → Markdown files, RAG injection → agentic Read/Grep/Glob)? Compensate in a skill/agent instruction. Examples already in place: page-writer reads `[RELEVANT_SOURCE_FILES]` itself, rewrites `#page-anchor` links to sibling `<page-id>.md` files, fills empty `Sources: [...]()` hrefs with `../../file#L10-L20` while keeping the visible text exact.
- Defect in the **original prompt text itself**? Preserve it.
- Every such compensation must be listed in `plugin/README.md` (the "Web renderer → plain files" bullet and the provenance table), so the verbatim claim stays auditable.

## Verifying verbatim fidelity

If anything under `plugin/templates/` is touched, re-verify by character-level diff against the **live** sources:

- `templates/wiki-page.md` and `templates/wiki-structure-*.md` ← template literals in `src/app/[owner]/[repo]/page.tsx` (`generatePageContent`, `determineWikiStructure`); unescape `` \` `` → `` ` ``, resolve the `isComprehensiveView` ternaries per variant, map the language ternary → `${language_name}` and the filePaths expression → `${file_list}`.
- `templates/ask.md` ← the simple-chat `system_prompt` f-string in `api/websocket_wiki.py`. Use `websocket_wiki.py`, **not** `api/prompts.py` — the two drift and the websocket versions are what actually ran.

The only tolerated diff is a trailing POSIX newline at end of file.

## Frozen upstream code

`src/`, `api/`, `tests/`, `test/`, the `README.*.md` translations, Dockerfiles, and compose files are the original web app, kept unmodified as the provenance for the templates. Do not refactor, lint, upgrade, or delete them as a side effect of other work. (A deliberate future "phase 2" may remove them wholesale — that's an explicit decision for the repo owner, not cleanup.)

## Distribution

- The marketplace manifest lives in [nijesmik/plugins](https://github.com/nijesmik/plugins) (git-subdir source → this repo's `plugin/` path). Do not re-add a `.claude-plugin/marketplace.json` here.
- Install: `/plugin marketplace add nijesmik/plugins` → `/plugin install deepwiki@nijesmik`. Only `plugin/` ships; keep it free of dead files (unused templates get deleted, not orphaned).
- Plugin naming: skills are invoked as `/deepwiki:<skill>`, agents referenced as `deepwiki:<agent>`.

## Layout

- `plugin/skills/{wiki,update,ask,deep-research}/SKILL.md` — orchestration (ours to change; `update` has no upstream counterpart — the original only offered cache-delete + full regeneration)
- `plugin/agents/page-writer.md` — per-page writer wrapper (ours to change)
- `plugin/templates/` — verbatim prompts (frozen, see above)
- `plugin/README.md` — design, install, provenance table; update it whenever wrappers or templates change
