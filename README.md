# Hi, I'm Randika 👋

I help teams migrate apps off retiring AI models before they break.

OpenAI is shutting down **GPT-4o, GPT-4o mini, GPT-4.1 and GPT-4.1 mini** in the API on **January 14, 2027**. After that date, any app that still calls them will fail. I find every place an app depends on the old model, move it to the recommended replacement, test it, and document the change.

## Migration case studies

### 1. Basic-Rag: PDF question-answering app (Python, LangChain)

- **Problem:** The app answered questions with `gpt-4o-mini`, which will stop working on Jan 14, 2027.
- **What I found:** The replacement, `gpt-6-luna`, is a reasoning model, and some reasoning models reject a `temperature` setting. I checked what the app's locked LangChain version sends for the new model: no `temperature`, so no package upgrade was needed.
- **Fix:** Switched the model to `gpt-6-luna`, fixed outdated "GPT-4" mentions in the README, and wrote a MIGRATION.md. Tested against a mock OpenAI server.
- **Links:** [My fork](https://github.com/prasadrandika/Basic-Rag/tree/migrate-off-gpt-4o-mini) · [Pull request (open)](https://github.com/gurezende/Basic-Rag/pull/2)

### 2. youtube_transcription_api: punctuates YouTube transcripts (Python, Flask)

- **Problem:** The API used `gpt-4o-mini` both to format text and, through `tiktoken`, to count tokens when splitting long transcripts.
- **What I found:** ⚠️ **Just changing the model name would have crashed the app.** `tiktoken.encoding_for_model("gpt-6-luna")` raises `KeyError`, in both the pinned version (0.7.0) and the latest (0.14.0), so upgrading doesn't help.
- **Fix:** Try the new model first, then fall back to `o200k_base`, the encoding the old model used. Chunk sizes stay exactly the same. The model name now lives in one setting. Tested end to end against a mock OpenAI server: 7 chunks, every request used the new model, and no words were lost.
- **Links:** [My fork](https://github.com/prasadrandika/youtube_transcription_api/tree/migrate-off-gpt-4o-mini) · [Pull request (open)](https://github.com/thinkingkoala/youtube_transcription_api/pull/2)

## How I work

1. Confirm the replacement on OpenAI's [official deprecations page](https://developers.openai.com/api/docs/deprecations).
2. Find every place that uses the old model: code, tokenizers, config, docs.
3. Make the smallest safe change, and only upgrade packages when needed.
4. Test it, and say clearly what was and wasn't tested.
5. Write a plain-English MIGRATION.md for the team.

**Tools:** Python · JavaScript · OpenAI API · LangChain · tiktoken · Claude Code
