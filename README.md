# ask-opus

Bring Opus judgment to Codex.

`ask-opus` is a Codex skill that asks Claude Opus for a focused second opinion from inside a Codex session.

It keeps the main workflow in Codex and calls Opus only when you explicitly invoke `$ask-opus`.

## Why

Claude Code is useful for project work, and Opus-level review can help before risky decisions.

Running the entire development session in Claude can burn through usage quickly. `ask-opus` lets Codex do the day-to-day work and call Opus only for the parts where stronger judgment is worth the cost.


## Installation

Paste this into a Codex session:

```text
$skill-installer --repo Jeon-wodn/ask-opus --path skills/ask-opus
```

Then restart Codex.

## Requirements

- [Codex CLI](https://github.com/openai/codex)
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)

Check that Claude CLI works:

```bash
claude --version
claude auth status --text
```

## Usage

Inside Codex:

```text
$ask-opus
```

Codex will compile the relevant task context, call Claude Code in non-interactive mode with tools disabled, and summarize the result as **Advisor (Opus)**.

The skill does not run automatically. Use it when Codex should pause before moving forward:

- before implementation
- before choosing an architecture
- after a risky change
- before finalizing a task
- when you want a second opinion


## Examples

Before implementation:

```text
Codex investigates the codebase and proposes a plan.
Run $ask-opus before letting Codex write code.
```

Before finalizing:

```text
Codex finishes the task and says it is done.
Run $ask-opus to check for missed risks, edge cases, or verification gaps.
```

## Model override

By default, `ask-opus` uses Claude Code's `opus` model alias.

Set `CLAUDE_OPUS_MODEL` to use a specific Claude Code model alias or model ID:

```bash
export CLAUDE_OPUS_MODEL="<opus-alias-or-model-id>"
```
