---
name: ask-opus
description: Use when the user explicitly invokes $ask-opus to ask Claude Opus for a second opinion during a Codex task, especially before architecture choices, risky changes, or final completion checks. Do not use implicitly for routine orientation, trivial tasks, or without user intent because it may send selected project context to Claude.
---

# Advisor (Claude Opus)

## Scope

Use only after the user explicitly invokes `$ask-opus` or asks for an Opus/Claude second opinion.

Do not call this automatically. This skill can send selected task context, code, and errors to Claude.

## Preconditions

Before the first advisor call in a Codex session, verify:
- `claude` CLI is installed (`claude --version`)
- Claude Code is authenticated (`claude auth status --text`)
- the Opus model in `_opus_model` is accessible by running the advisor call

If either condition fails: surface the error to the user, do not proceed.

After `claude --version` and `claude auth status --text` both succeed once, remember that preflight is satisfied for the rest of the current Codex session. Do not rerun those two checks on later `$ask-opus` calls in the same session unless `CLAUDE_OPUS_MODEL` changes or an advisor call fails with a CLI, auth, quota, or model-access error.

## What Opus Cannot See

The advisor command disables Claude Code tools. Treat Opus as having no useful prior access to:
- Conversation history
- The current Codex session state
- Files unless you paste their relevant contents

Compile everything it needs inline. Paste only the relevant file snippets and command outputs.

## Context to Compile

Do not ask the user. Compile yourself from the current session.

Before including content:
- Exclude secrets, tokens, credentials, private keys, and unrelated proprietary material.
- Prefer minimal snippets over whole files.
- If sensitive context is essential, stop and ask the user before sending it.

**In long sessions, preserve and trim as follows:**
- Preserve: original task verbatim, all error messages verbatim, current file contents relevant to the decision, candidate options
- Trim: successful steps that didn't affect the current state, repetitive tool outputs, already-resolved sub-problems

Include:
- **Task** — original goal, verbatim if possible
- **History** — every significant step with concrete results; paste exact error text, do not summarize failures
- **Current state** — relevant file paths, line numbers, code snippets pasted inline (not described)
- **Decision** — the specific question, with candidate options and what distinguishes them
- **Stakes** — what action would differ depending on the answer

## Procedure

1. If declaring complete: **save all files and results first**, then proceed.

2. Compile the context above. Write it to a temp file to avoid shell escaping issues:

```bash
_opus_model="${CLAUDE_OPUS_MODEL:-opus}"
_ctx=$(mktemp /tmp/ask-opus-XXXXXX.md)
cat > "$_ctx" << 'EOF_ASK_OPUS'
You are a stronger reviewer. The agent compiled the following from the full session transcript because you have no access to conversation history, filesystem, or tools. Identify what is load-bearing, what may have been missed, and what the agent should do next.

[COMPILED CONTEXT GOES HERE — replace this line with actual compiled context before running]

Respond in under 100 words. Use enumerated steps, not explanations.
EOF_ASK_OPUS

claude -p --model "$_opus_model" --max-turns 1 --tools "" --no-session-persistence --disable-slash-commands < "$_ctx"
_status=$?
rm -f "$_ctx"
exit "$_status"
```

Replace `[COMPILED CONTEXT GOES HERE — replace this line with actual compiled context before running]` with the actual compiled context before executing.

3. **Failure handling**: if `claude -p` fails (auth error, network, quota, model not supported), show the raw error to the user and stop. Do not fabricate advice or proceed as if the call succeeded.

4. **Give the advice serious weight.** If you follow a step and it fails empirically, or primary-source evidence contradicts the advice (file says X, output shows Y), adapt. A passing self-test is not evidence the advice is wrong — it's evidence your test doesn't check what the advice is checking.

5. If you've already retrieved data pointing one way and the advisor points another: don't silently switch. Surface the conflict in one more advisor call — "I found X, you suggest Y, which constraint breaks the tie?" The advisor saw your evidence but may have underweighted it; a reconcile call is cheaper than committing to the wrong branch.

6. Summarize **actionable points** labeled **Advisor (Opus)**. If the user asks for the full response, show it.
