---
name: "muyi-translate"
description: "Use when the user asks to translate or localize an article, document, URL, Markdown file, or inline text between languages, especially when terminology consistency, long-document chunking, or publication-quality refinement matters. Supports quick, normal, and refined modes plus EXTEND.md preferences and glossary loading."
---

# Muyi Translate

## When to use
- Translate articles, documents, blog posts, README files, URLs, or inline text.
- Localize Markdown content while preserving structure, links, and code blocks.
- Handle long documents where chunking and terminology consistency matter.
- Produce either a fast draft or a refined publication-quality translation.

## Resource map
- `references/config/extend-schema.md`: EXTEND.md fields and glossary schema
- `references/config/first-time-setup.md`: first-run question set and EXTEND.md template
- `references/workflow-mechanics.md`: source materialization, output directory, conflict handling
- `references/refined-workflow.md`: analysis, critique, revision, and polish guidance
- `references/subagent-prompt-template.md`: shared-context template for chunked translation
- `references/glossary-en-zh.md`: built-in EN -> ZH glossary

## Runtime
- Prefer `bun` for the chunking script.
- If `bun` is unavailable but `npx` exists, use `npx -y bun`.
- If neither is available, explain that automated chunking is unavailable and either install Bun or chunk manually.

## Preference lookup
Check for `EXTEND.md` in this exact order and stop at the first match:

1. `.muyi-skills/muyi-translate/EXTEND.md`
2. `${XDG_CONFIG_HOME:-$HOME/.config}/muyi-skills/muyi-translate/EXTEND.md`
3. `$HOME/.muyi-skills/muyi-translate/EXTEND.md`

If a file is found, read it and apply it. On first use in a session, briefly tell the user which file is active.

If no file exists, do not silently assume a persistent profile. Before translating, ask once for target language, mode, audience, style, and save location, then create `EXTEND.md`. If the user explicitly says to skip setup, proceed with built-in defaults for this run only and state that no preferences were saved.

## Defaults
- `target_language`: `zh-CN`
- `default_mode`: `normal`
- `audience`: `general`
- `style`: `storytelling`
- `chunk_threshold`: `4000`
- `chunk_max_words`: `5000`

## Workflow
1. Load preferences and merge glossary sources.
   - Load the built-in language-pair glossary when available.
   - Merge `EXTEND.md` inline `glossary`.
   - Merge `EXTEND.md` `glossary_files`, resolved relative to the `EXTEND.md` file.
   - Merge any glossary override provided for the current run.
   - Read `references/config/extend-schema.md` only if field details are needed.
2. Materialize the source.
   - File path: use as-is.
   - URL or inline text: save it to `translate/{slug}.md` first.
   - Create the output directory as `{source-dir}/{source-basename}-{target-lang}/`.
   - Follow `references/workflow-mechanics.md` for naming and conflict handling.
3. Select the mode.
   - Quick: translate directly.
   - Normal: analyze, assemble context, then translate.
   - Refined: analyze, draft, critique, revise, then polish.
   - If the user says `快翻` or `quick`, use quick mode.
   - If the user says `精翻`, `refined`, `publication quality`, or asks to continue polishing, use refined mode.
   - Otherwise use the configured default.
4. Assess content length.
   - Quick mode never chunks.
   - Normal and refined modes translate as a single unit below `chunk_threshold`.
   - At or above the threshold, extract terminology first, then chunk the Markdown.
5. Chunk long Markdown with `scripts/main.ts`.

```bash
bun scripts/main.ts <file> --max-words <n> --output-dir <output-dir>
```

If only `npx` is available:

```bash
npx -y bun scripts/main.ts <file> --max-words <n> --output-dir <output-dir>
```

This writes chunk files under `<output-dir>/chunks/`.

6. For chunked translation, create shared context in `02-prompt.md` using `references/subagent-prompt-template.md`.
   - Translate chunks sequentially by default.
   - Only use worker agents in parallel if the user explicitly asked for delegation or sub-agents and the chunks are independent.
   - Merge translated chunks in order.
   - If `chunks/frontmatter.md` exists, prepend it before saving the merged result.
7. Apply these translation principles.
   - Preserve meaning, facts, Markdown structure, links, and code blocks.
   - Translate for natural target-language flow, not literal word order.
   - Keep terminology consistent and annotate first occurrences only when useful.
   - Preserve figurative meaning and emotional tone, even if wording must change.
   - Add concise translator's notes only when the target audience truly needs them.
   - If YAML frontmatter exists, rename source metadata fields with a `source` prefix and add translated top-level text fields where appropriate.
8. Save outputs with stable filenames.
   - Quick: `translation.md`
   - Normal: `01-analysis.md`, `02-prompt.md`, `translation.md`
   - Refined: `01-analysis.md`, `02-prompt.md`, `03-draft.md`, `04-critique.md`, `05-revision.md`, `translation.md`
9. After the final translation, do a lightweight image-language check.
   - If a referenced image likely still contains source-language text, tell the user as a plain list.
   - Do not localize images unless the user asks.

## Mode details
### Quick
Translate directly to `translation.md`. Use this for short or disposable content.

### Normal
Create `01-analysis.md` and `02-prompt.md`, then write `translation.md`.

If the user later asks to continue polishing, rename the current `translation.md` to `03-draft.md` and continue with critique, revision, and polish using `references/refined-workflow.md`.

### Refined
Follow the review loop in `references/refined-workflow.md`.

Use the main agent for critique, revision, and polish even if the initial draft was chunked.

## Validation
- Ensure the final output preserves Markdown structure and stable file naming.
- Re-read the translation for terminology drift, broken links, code block corruption, and inconsistent annotation depth.
- If `EXTEND.md` creation or chunk automation was skipped because tooling was missing, state that explicitly.
