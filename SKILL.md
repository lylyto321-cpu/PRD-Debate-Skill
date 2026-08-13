---
name: prd-debate
description: Runs a structured, multi-round debate between Claude (host and proposer) and OpenAI Codex (reviewer, via the codex CLI) to turn a rough product idea or a pile of scattered UX/feature complaints into a well-reasoned PRD -- product positioning, ideal state, a dimensional gap analysis, and a phased strategy -- instead of the shallow "here are N independent fixes for your N complaints" a single AI pass tends to produce. Use this whenever the user wants to write a PRD, scope a feature, define product requirements, resolve a list of user-reported issues or bugs into a coherent plan rather than patching each one separately, mentions wanting Codex or another model to challenge a product proposal, or says things like "help me think through this feature properly" or "I don't want a band-aid for each of these, I want to understand what's actually going on."
---

# PRD Debate

## Why this exists

Handed six independent UX complaints, a single model tends to hand back six independent patches -- one fix per symptom, no shared diagnosis. The fix is structural: force an adversarial second model to review the product definition itself, layer by layer, before scope or fixes get discussed at all. This is the same underlying mechanism as the general-purpose `debate` skill, specialized for product requirements work -- if this skill isn't installed for some reason, the `debate` skill's four layers map directly onto PRD work and can be used instead.

## Setup: confirm Codex is available

```bash
command -v codex
```

If this fails, tell the user the OpenAI Codex CLI needs to be installed and authenticated -- give them the concrete steps, don't just say "install it":

```bash
npm install -g @openai/codex   # NOT `npm i -g codex` -- that unscoped package is an unrelated 2012 project
codex login                    # opens a browser to sign in with ChatGPT, or use --with-api-key
codex --version                # verify
```

Then stop. Do not simulate the debate by arguing against yourself as both proposer and reviewer -- an independent model catching Claude's blind spots is the entire point, and self-review quietly forfeits that while looking like it worked.

`scripts/ask_codex.sh` pipes the prompt to `codex exec -` (reads the prompt from stdin, runs to completion, prints the reply to stdout) and already includes the install message above -- just run the script and relay its stderr if it fails.

## The four layers, for product work

1. **Problem Definition** -- what's actually going on, not the list of symptoms the user handed you. If the user came in with N bug reports or complaints, your job here is to find what they have in common, not to restate them.
2. **Ideal State** -- define the product structurally: the user journey, and the handful of core product attributes/entities that everything else hangs off of. Watch for the trap this layer usually falls into: producing a capability list ("it should let users do X, Y, Z") that *sounds* like a definition but isn't one. Push until you can answer things like: what are the core entities and how do they relate to each other; what states does a unit of work move through over its lifecycle; what's the boundary between this feature and the rest of the system. A real example from this mechanism's origin: a proposal describing five product attributes got sent back by the reviewer for still lacking three more basic definitions -- how the feature's workspace relates to the main system, the lifecycle states of a "session," and the object model distinguishing one-shot inputs from long-lived project assets. That's the level of structural precision this layer needs before gap analysis can mean anything.
3. **Gap Analysis** -- compare ideal state to current state across explicit dimensions, not a re-listing of the original complaints. A useful default set of dimensions (adapt to the actual product, don't force these blindly):
   - **Boundaries & ownership** -- are entities, subsystems, and their responsibilities clearly separated, or is something doing two jobs?
   - **Lifecycle** -- does the relevant unit of work actually support the states it needs (create, pause, resume, complete), or is it an unbounded, unmanaged stream?
   - **Visibility & control** -- can the user see the state they're in and steer it, or are they flying blind?
   - **Information architecture** -- is there one clear primary surface, or has structure accreted through ad hoc additions?
4. **Strategy** -- a phased plan, not a flat backlog. Classify every candidate move:
   - **No-regret** -- safe to ship now regardless of what the foundational work later requires. The test: does it *not* depend on the foundational piece, and does it *still hold* once that foundational piece lands? Both conditions, not just one.
   - **Foundational** -- has to exist before anything built on top of it is real (e.g. the core lifecycle/object model from layer 2).
   - **Deferred / explicitly not now** -- name these, don't just omit them. Team capacity and timeline questions ("2 sprints or 3 releases") come *after* this classification is settled, not before -- resist letting scheduling anxiety collapse the sequencing logic.

## Running a turn

1. **Draft** -- as Proposer, write Claude's position for the current layer.
2. **Compose the reviewer prompt** -- role framing (Codex is the adversarial-but-constructive Reviewer, should hunt for missing definitions and premature conclusions, not just agree), a short recap of what prior layers confirmed, this layer's goal, and the latest proposal. End with an explicit ask: name what's missing and why it matters now, or state plainly the layer is converged.
3. **Call Codex**:
   ```bash
   scripts/ask_codex.sh <<'EOF'
   <the composed prompt>
   EOF
   ```
4. **Log the turn** using `references/turn_format.md`.
5. **Branch** -- structural pushback: revise and repeat. Sign-off, or round cap hit (default 5 per layer): advance, carrying forward a short recap.

If Codex's pushback turns repetitive or fixates on something cosmetic rather than structural, note the disagreement and move on -- the mechanism exists to catch blind spots, not to hand Codex veto power over every layer.

## Retrospective verification

Before locking the Strategy layer, take every deferred or trimmed item and check it against what layers 1-3 already committed to. Put this check to Codex directly rather than answering it yourself: "does cutting this quietly break something we already agreed mattered?" Self-grading your own trims is exactly the blind spot this mechanism exists to avoid.

## Output

Produce the full turn-by-turn transcript, plus a final PRD following `assets/prd_template.md` (product positioning / ideal state / dimensional gap analysis / phased strategy). Ask the user where to save these (default: current working directory, e.g. `<feature-slug>-debate.md` and `<feature-slug>-prd.md`). Conduct the debate in whatever language the user is working in.
