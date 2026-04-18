# Strategy Consultant (a.k.a. Worker)

A Cowork plugin that performs the analytical work of a strategy consultant or any knowledge worker. Takes a business problem from definition through research, validation, synthesis, and delivers an executive-grade deliverable.

## Entry point

`/engagement` (backed by the `engagement-manager` skill) is the **singular entry point** to this plugin. It orchestrates problem-definition → research → sense-check → synthesis → client-report as internal phases. The other three commands — `/research`, `/define-problem`, and `/interview-guide` — are **continuation-only**: they run when an engagement workspace (`./engagement/`) already exists on disk. If an agent pattern-matches a verb in a user's request to one of these sub-commands without a workspace present, the sub-command must stop and hand control back to `/engagement`, which routes to the correct starting phase. Do not invoke sub-commands cold, and do not attempt to reconstruct upstream artifacts (Precision Anchor, Client Question Checklist, Source Material Extraction Log, data-source answers) inside a sub-command — that rebuilds the orchestrator piecemeal.

## Who It's For

- **Management Consultants** — Strategy and generalist consultants at firms who want to accelerate the research-to-deliverable cycle
- **Corporate Strategy Teams** — In-house strategists running internal engagements for C-suite
- **Corporate Development / M&A** — Teams scoping market entries, acquisitions, and competitive landscapes
- **Independent Advisors / Boutique Firms** — Solo or small-team consultants who need the analytical horsepower of a larger team
- **Any knowledge work** - Any work that requires similar skill

## How It Works

This plugin is designed to handle 80–90% of the analytical workload in a strategy engagement — research, evidence validation, structured synthesis, and report drafting — so that the consultant can focus on the 10–20% that creates unique value: client relationship judgment, proprietary insight, and final recommendations. All outputs are intended as high-quality working drafts for expert review, not finished deliverables.

## What It Does

This plugin runs a complete consulting-grade analytical workflow:

1. **Define the problem** — Sharpens a vague business challenge into a decision-oriented question with clear scope
2. **Structure hypotheses** (optional) — Builds a testable hypothesis tree when the problem benefits from structured decomposition
3. **Research** — Deploys two independent research agents in parallel, then validates findings with a third agent for consistency and source quality
4. **Sense-check** — Pressure-tests all findings through triangulation, steel-manning the counter-argument, math checks, and bias scanning
5. **Synthesize** — Builds a storyline that directly answers the client question, organized by the logic of the answer (not a rigid framework)
6. **Deliver** — Produces an executive-grade Word document with clear language, sourced claims, and zero consulting clichés

## Commands

| Command | Purpose |
|---------|---------|
| `/engagement <topic>` | Run the full end-to-end consulting engagement |
| `/research <topic>` | Deploy three research agents to investigate a topic standalone |
| `/define-problem <brief>` | Sharpen a vague question into a decision-oriented problem statement |
| `/interview-guide <topic>` | Plan expert interviews and create structured guides |

## Skills

| Skill | Purpose |
|-------|---------|
| **engagement-manager** | Orchestrator — runs the full end-to-end workflow |
| **problem-definition** | Scopes and sharpens the client question |
| **hypothesis-tree** | Builds MECE hypothesis trees (optional, when it adds value) |
| **research** | Dispatches 3 agents for parallel research and validation |
| **sense-check** | Pressure-tests findings for robustness |
| **synthesis** | Builds the client-facing storyline and argument |
| **client-report** | Produces the final .docx deliverable |
| **expert-interview** | Plans, guides, and processes expert interviews |

## Agents

| Agent | Role |
|-------|------|
| **analyst-alpha** | Independent research thread A — direct angle (market data, financials, benchmarks) |
| **analyst-bravo** | Independent research thread B — complementary angle (competitive dynamics, case studies, contrarian evidence) |
| **research-validator** | Cross-checks both analysts for consistency, source quality, and gaps |

## Connectors (Optional)

This plugin works without any external integrations. See `CONNECTORS.md` for optional connectors (document storage) that enhance team workflows.

## Design Principles

- **Delivery mode only** — produces outputs directly, no Socratic coaching
- **Evidence-traced** — every claim in the final report traces back to a sourced finding
- **Bias-resistant** — two independent research threads + validation reduces confirmation bias
- **No forced frameworks** — the storyline structure follows the logic of the answer
- **No clichés** — a comprehensive banned-language list ensures clear, direct executive writing
- **Expert-in-the-loop** — designed as a 80–90% accelerator, not a replacement for professional judgment

## Entry-point doctrine for workflow plugins

This section generalises the architecture used above so it is reusable when adding new skills to this plugin or when authoring other multi-phase workflow plugins.

**The pattern**
- **One orchestrator as the singular entry.** For a multi-phase workflow, exactly one skill (and its command) is the entry point. In this plugin: `engagement-manager` / `/engagement`.
- **Sub-skills as continuation subroutines.** Every other command in the workflow runs only as a continuation of an existing workspace. It checks for the workspace on disk; if absent, it stops and routes back to the orchestrator.
- **Gates owned by the skill that needs them.** Pre-flight gates (data-source inquiry, clarifying questions, etc.) live in the skill that needs the information, not in the entry-point command and not duplicated across siblings. They fire exactly once, in order, because the orchestrator invokes the skills in sequence.
- **A single workspace folder as the continuation test.** The orchestrator creates and maintains one well-known folder (here: `./engagement/`). Sub-commands check for its presence to decide whether to continue or hand control back to the orchestrator.

**Why this matters.** When sub-commands appear as peers of the orchestrator on the command surface, agents pattern-match a verb in the user's request to a sub-command and skip the orchestrator — silently dropping problem definition, sense-check, synthesis, and the real delivery skill. Making the orchestrator the singular entry and demoting sub-commands to continuation-only preserves the phase sequence.

**Things to avoid**
- Do not bolt pre-flight gates onto each sub-command. That rebuilds the orchestrator piecemeal.
- Do not remove sub-commands. They remain useful for continuations and for users who know exactly where they are in the workflow.
- Do not weaken the enforcement language inside any skill's `SKILL.md` — the gates are authoritative.

**Checklist when adding a new skill to this or any workflow plugin**
1. Does the skill belong inside an existing phase, or is it a new phase? If new, wire it into the orchestrator's phase list.
2. If it also needs a standalone command, mark that command `CONTINUATION COMMAND — do not use as a fresh entry point` at the very top (after frontmatter) and have it check for `./engagement/` before any other tool call.
3. Put gates inside the skill's `SKILL.md`, not in the command file.
4. Add a leading sentence to the skill's `description:` frontmatter: "Internal phase of the strategy-consultant engagement — invoke via engagement-manager, not directly."
5. Update this README's command table and phase sequence.
