# prd-debate

A [Claude Code skill](https://docs.claude.com/en/docs/claude-code/skills) that turns a rough feature idea or a pile of scattered bug reports/UX complaints into a proper PRD, by having **Claude propose** and **OpenAI Codex review** across four gated layers: Problem Definition → Ideal State → Gap Analysis → Strategy.

The idea: hand a single model six independent UX complaints and it tends to hand back six independent patches -- one fix per symptom, no shared diagnosis. An independent second model, reviewing the product definition itself layer by layer before scope gets discussed at all, catches the missing structural definitions that the author glides past mid-flow.

This is the product-focused sibling of the general-purpose [`debate`](https://github.com/<your-username>/debate-skill) skill -- same underlying mechanism, tuned for PRD work specifically. If you only need one, `debate`'s four layers map directly onto PRD work too.

## What it does

- **Ideal State** pushes for structural definitions (entities, relationships, lifecycle states) instead of a capability list that just sounds like one
- **Gap Analysis** defaults to four dimensions -- boundaries & ownership, lifecycle, visibility & control, information architecture -- adapted to the actual product rather than a flat re-listing of complaints
- **Strategy** classifies every candidate move into no-regret (ship now) / foundational (must exist first) / deferred (named, not just omitted), with team-capacity questions deliberately answered *after* that classification, not before
- Runs a retrospective check before locking in the final strategy, mapping every deferred/trimmed item back against earlier commitments
- Logs a full turn-by-turn transcript alongside the final PRD
- Refuses to fake the debate solo if the Codex CLI isn't available -- an independent model's blind spots not overlapping with Claude's is the entire point

## Prerequisites

The [OpenAI Codex CLI](https://developers.openai.com/codex) must be installed and authenticated:

```bash
npm install -g @openai/codex   # NOT `npm i -g codex` -- that unscoped package is an unrelated 2012 project
codex login                    # opens a browser to sign in with ChatGPT, or use --with-api-key
codex --version                # verify
```

The skill checks for `codex` on `PATH` before starting and prints these exact steps if it's missing, so this isn't a hard blocker to installing the skill itself -- just to actually running a debate.

## Installing

```bash
git clone https://github.com/<your-username>/prd-debate-skill.git
cp -r prd-debate-skill ~/.claude/skills/prd-debate
```

Claude Code picks up skills from `~/.claude/skills/<name>/SKILL.md` automatically -- no restart needed for a fresh session. To install for one project only, copy into `<project>/.claude/skills/prd-debate` instead. The destination directory name (`prd-debate`) is what Claude Code uses to key the skill, so keep it as `prd-debate` even though the GitHub repo is named `prd-debate-skill`.

## Using it

Just ask, in whatever words fit the moment -- this triggers on intent, not on exact phrasing:

- *"I've got six UX complaints on this feature and I don't want six band-aids -- help me write a real PRD."*
- *"Help me scope this feature properly, and have Codex push back on it."*
- *"Turn these bug reports into a proper requirements doc, not a patch list."*

Each debate produces two files (path/naming is asked at the end, defaulting to the current directory):
- `<slug>-debate.md` -- the full turn-by-turn transcript, including where Codex pushed back and why
- `<slug>-prd.md` -- the final PRD (product positioning / ideal state / dimensional gap analysis / phased strategy)

## How a layer actually converges

A layer ends when Codex explicitly signs off, not when Claude feels done. A real turn from this mechanism's origin, where a proposal describing five product attributes got sent back for still lacking basic structural definitions:

> Agree with the direction so far, but the priority right now isn't "how," it's pinning down what the workspace actually *is* on the product level... What's missing isn't more features, it's a few more basic definitions -- without them, later discussion drifts without anyone noticing.

See [`references/turn_format.md`](references/turn_format.md) for the full worked example and the transcript format this skill uses.

## Design notes

- **Why a script instead of inline `codex` calls**: `scripts/ask_codex.sh` pipes the composed review prompt to `codex exec -` (reads from stdin, avoids shell-quoting issues with long multi-line prompts) and centralizes the "Codex isn't installed" error message so it's consistent and actionable.
- **Why Codex can't be faked**: `SKILL.md` explicitly instructs against having Claude argue both sides if Codex is unavailable. A solo debate looks like it worked but silently throws away the one thing this mechanism buys you.
- **Why round caps exist**: unbounded review loops are a real failure mode. The default cap (5 rounds/layer) is a judgment call, not a hard rule -- the skill explicitly tells Claude to use judgment and move on if Codex's pushback turns repetitive or cosmetic rather than structural.

## Credit

The debate mechanism itself is adapted from a product manager's write-up on using adversarial multi-model review to avoid "quality degradation in single-session analysis of complex problems." This skill is an independent implementation of that idea for Claude Code, not an official Anthropic or OpenAI product.

## License

MIT -- see [LICENSE](LICENSE).
