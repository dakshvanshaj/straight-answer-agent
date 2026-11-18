# How it works

*This page goes deep on purpose — it's the proof the homepage isn't just copy. The architecture below is real
and running; the source code isn't published, for the reasons on the [overview page](index.md).*

## The shape: plan → execute → narrate

The agent doesn't write an answer and hope it's right. For every question, it picks from a fixed set of tools,
runs them against the real data, and only then writes prose — and the prose is built from what the tools
actually returned, not from the model's own sense of what's probably true. Concretely:

1. **Plan.** The model reads the question and decides which tool(s) actually answer it — it can chain several
   in one turn to compare periods, connect a margin drop to a supplier, or build up a multi-step answer.
2. **Execute.** Each tool runs a real, verified lookup and returns a result built from the system's own data —
   never text the model invents.
3. **Narrate.** The model writes the final answer using only what the tools returned. It's shown the same
   numbers a person would see on screen, pre-rounded the way the screen rounds them, so the figure in the
   sentence always matches the figure in the table under it.

This is why a question the agent can't actually answer gets "I don't have that" instead of a confident guess —
there's no tool result to narrate from, so there's nothing to write.

## The tool registry: 42 tools, two kinds

- **34 read tools** — look things up: inventory position, cash at risk, supplier performance, what changed
  since yesterday, why a number is what it is. These can run freely; there's nothing to approve because nothing
  changes.
- **8 propose tools** — purchase orders, supplier/customer emails, a config change, an alert override, a line
  edit on an order. Every one of these builds a draft and stops. Nothing sends, saves, or executes until a
  human reviews it and says yes.

That split is enforced in exactly one place in the code — the single function every tool call passes through —
rather than repeated inside each of the 42 tools individually. A new tool can't accidentally ship able to act
on its own; it can only ship that way by someone explicitly deciding it should.

## The trust layer, in four parts

- **Inputs.** Questions are length-capped and stripped of control characters before they reach the model, so a
  malformed or oversized input can't inflate cost or smuggle something unexpected through.
- **Prompts.** Anything that didn't come from the system itself — the user's question, recalled memory, data
  from a connected ERP — is fenced off from instructions before the model sees it. The model is told, explicitly,
  which parts are data to report on and which parts are instructions to follow, which is the standard defence
  against a prompt-injection attempt hidden inside imported data.
- **Outputs.** Cite-or-omit: the model is structurally blocked from putting a number in its answer that isn't
  backed by a tool result. A wrong number in front of a customer is the one failure that costs the most trust,
  so this is the rule with zero exceptions.
- **Logs.** Raw prompts and responses are redacted out of the operational audit trail by default, so what a
  customer asked never ends up sitting in a log table outside their own isolated data.

## Memory, without sending your data anywhere

The agent remembers three things, all computed locally with a small, free embedding model — nothing is sent to
a third party to generate the memory itself:

- **Conversation recall** — prior questions and answers are recalled when a follow-up refers back to them
  ("the PO you just drafted," "what about last month instead").
- **Decisions** — when someone snoozes an alert, overrides a finding, or acts on a suggestion, that's recorded
  as a fact, not inferred later from guesswork.
- **Stated preferences** — anything the user explicitly tells it to remember.

A memory lookup failing never breaks an answer or blocks an action — it's a nicety layered on top of the real
system, never a dependency the core answer relies on.

## What's real and what isn't published

- **Real:** everything described on this page and the overview — it's the architecture actually running
  Narada OS, built and operated solo.
- **Not published:** the source code. It's proprietary production software for a real (if still customer-less)
  venture, not a template to copy-paste — see the note on the [overview page](index.md) for why.
- **What you'd actually get:** this exact pattern — plan/execute/narrate, the read/propose tool split, the
  four-part trust layer, grounded memory — rebuilt for your own data and your own tools, not a reused copy of
  Narada's code.
