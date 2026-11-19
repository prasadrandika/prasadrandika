# Hi, I'm Randika 👋

I help teams migrate apps off retiring AI models before they break.

OpenAI is shutting down **GPT-4o, GPT-4o mini, GPT-4.1 and GPT-4.1 mini** in the API on **January 14, 2027**. After that date, any app that still calls them will fail. I find every place an app depends on the old model, move it to the recommended replacement, test it, and document the change.

## Migration case studies

### 1. Basic-Rag: PDF question-answering app (Python, LangChain)

- **Problem:** The app answered questions with `gpt-4o-mini`, which will stop working on Jan 14, 2027.
- **What I found:** The replacement, `gpt-6-luna`, is a reasoning model, and some reasoning models reject a `temperature` setting. I checked what the app's locked LangChain version sends for the new model: no `temperature`, so no package upgrade was needed.
- **Fix:** I opened a PR switching the model to `gpt-6-luna`, tested against a mock OpenAI server. After that, the maintainer applied the model change himself ([`139efd3`](https://github.com/gurezende/Basic-Rag/commit/139efd3)), so my PR now only fixes the outdated "GPT-4" mentions in the README.
- **Links:** [My fork, with migration notes](https://github.com/prasadrandika/Basic-Rag/tree/migration-notes) · [Pull request (open)](https://github.com/gurezende/Basic-Rag/pull/2)

### 2. youtube_transcription_api: punctuates YouTube transcripts (Python, Flask)

- **Problem:** The API used `gpt-4o-mini` both to format text and, through `tiktoken`, to count tokens when splitting long transcripts.
- **What I found:** ⚠️ **Just changing the model name would have crashed the app.** `tiktoken.encoding_for_model("gpt-6-luna")` raises `KeyError`, in both the pinned version (0.7.0) and the latest (0.14.0), so upgrading doesn't help.
- **Fix:** Try the new model first, then fall back to `o200k_base`, the encoding the old model used. Chunk sizes stay exactly the same. The model name now lives in one setting. Tested end to end against a mock OpenAI server: 7 chunks, every request used the new model, and no words were lost.
- **Links:** [My fork](https://github.com/prasadrandika/youtube_transcription_api/tree/migrate-off-gpt-4o-mini) · [Pull request (open)](https://github.com/thinkingkoala/youtube_transcription_api/pull/2)

### 3. dsl: ProjectDiscovery's DSL engine (Go)

- **Problem:** The `llm_prompt` helper defaulted to `gpt-4o-mini`.
- **What I found:** The shared client in `projectdiscovery/utils` always sent `temperature: 0`, which may not work with the newest reasoning models. The maintainer confirmed it was a real problem.
- **Outcome:** I opened an issue first. The maintainer fixed the temperature problem in utils v0.11.15 and opened a PR with my commit plus the utils bump.
- **Links:** [Issue #381 (open)](https://github.com/projectdiscovery/dsl/issues/381) · [Pull request #382 (open)](https://github.com/projectdiscovery/dsl/pull/382)

## More migration work

- **WeClaw** (Go): HTTP agents fell back to `gpt-4o-mini`. A PR with a test: [#88 (open)](https://github.com/fastclaw-ai/weclaw/pull/88)
- **Nasiko** (Go, Rust, Python): example agents and the LLM router default to `gpt-4o-mini`, and `gpt-6-luna` needs `reasoning_effort: "none"` for tool calls on Chat Completions. An issue to agree the scope: [#369 (open)](https://github.com/Nasiko-Labs/nasiko/issues/369)
- **openrisk** (Go, ProjectDiscovery): hard-codes `gpt-4o`. An issue with a ready branch: [#118 (open)](https://github.com/projectdiscovery/openrisk/issues/118)

## luna-check

I turned the problems from these migrations into **[luna-check](https://github.com/prasadrandika/luna-check)**, a small CLI and GitHub Action that finds code that will break at the shutdown: retiring model IDs, `tiktoken.encoding_for_model()` calls, and tool calls sent without `reasoning_effort`.

📝 Article: [GPT-4o shuts down on Jan 14, 2027: what breaks when you switch to gpt-6-luna](https://dev.to/prasadrandika/gpt-4o-shuts-down-on-jan-14-2027-what-breaks-when-you-switch-to-gpt-6-luna-59kg)

## How I work

1. Confirm the replacement on OpenAI's [official deprecations page](https://developers.openai.com/api/docs/deprecations).
2. Find every place that uses the old model: code, tokenizers, config, docs.
3. Make the smallest safe change, and only upgrade packages when needed.
4. Test it, and say clearly what was and wasn't tested.
5. Write a plain-English MIGRATION.md for the team.

**Tools:** Python · Go · JavaScript · OpenAI API · LangChain · tiktoken · Claude Code
