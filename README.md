# OpSpec — Operational Specification Language

OpSpec is a general-purpose specification language for operational workflows. Any
process that has stages, decision gates, human interaction points, and the capacity
to learn and adapt can be expressed as an OpSpec.

A coding agent reads an OpSpec and produces the executor's program — a Temporal
workflow, a GitHub Actions pipeline, a Python module, shell scripts, or whatever
the target executor requires. The OpSpec is the specification; the produced program
is the executable. A human can also follow the same OpSpec manually, step by step.

OpSpec is an **independent specification language**. It can pair with NLSpec (a
specification standard for what to build) for software delivery workflows, but
OpSpec stands on its own. A claims processing OpSpec, a patient intake OpSpec,
or a supply chain OpSpec has no NLSpec dependency.

---

## What's in This Directory

| File | Purpose |
|------|---------|
| `OPSPEC-TEMPLATE.md` | **The OpSpec language definition.** Primitives (STAGE, GATE, BINDING, LEARNING RULE, APPROVAL, INTERACTION, FEEDBACK_RULE), section structure, executor requirements, binding types, learning governance, scenario conventions, and worked examples. Start here to write an OpSpec. |

---

## Quick Start

1. Read `OPSPEC-TEMPLATE.md` — it is both the language reference and a fill-in template
2. Copy the template and fill in your workflow's stages, gates, bindings, and learning rules
3. Hand the completed OpSpec to a coding agent to produce the executor program

---

## Key Concepts

**STAGE** — A phase of work with entry/exit conditions and concrete operations.

**GATE** — A three-band decision point (PASS / REWORK / FAIL) between stages.

**BINDING** — An artifact template (prompt, rule, tool config) that the coding agent produces as part of the executor program.

**LEARNING RULE** — Governed self-modification: the workflow observes patterns, proposes changes to itself, and submits them for approval within declared boundaries.

**APPROVAL** — A human interaction point where the executor pauses and waits.

**INTERACTION** — A customer-facing touchpoint where the workflow communicates with an external party (customer, patient, applicant). Unlike APPROVAL (internal review), INTERACTION represents outbound communication that may await a response or collect feedback.

**FEEDBACK_RULE** — Satisfaction-driven graduation: the workflow observes customer feedback signals (ratings, reopens, escalation requests), accumulates positive patterns, and graduates them into deterministic rules that bypass LLM stages. The customer-facing counterpart to LEARNING RULE.

**Executor** — The system that runs the produced program. Can be a workflow engine (Temporal, Airflow), a CI/CD pipeline (GitHub Actions, Jenkins), a controller, or a shell runner.

---

## Applications

OpSpec applies to any operational workflow:

- Software delivery pipelines (code → build → test → deploy → monitor)
- Insurance claims processing (intake → verify → assess → adjudicate → pay)
- Loan origination (application → underwrite → approve → fund → service)
- Patient intake and triage (register → assess → route → treat → follow-up)
- Supply chain orchestration (order → source → manufacture → ship → deliver)
- Content moderation (receive → classify → review → act → appeal)
- Customer onboarding (sign-up → verify → provision → train → activate)
- Customer support (intake → classify → route → draft → review → feedback → graduate)

---

## Writing Your First OpSpec

1. **Read `OPSPEC-TEMPLATE.md`** — every section has syntax examples and conventions.

2. **Start with the Abstract** — one paragraph describing what the workflow does,
   its domain, inputs, outputs, and primary approval gates.

3. **Define your STAGEs and GATEs** — map out the workflow as a sequence of concrete
   stages with decision gates between them.

4. **Add BINDINGs** — for each stage that needs artifacts (prompts, rules,
   tool configs), write a BINDING template with runtime variables.

5. **Declare APPROVALs** — identify where humans enter the loop. Every APPROVAL
   has a channel, SLA, and escalation path.

6. **Add INTERACTIONs** — if the workflow communicates with external parties
   (customers, patients, applicants), declare INTERACTION blocks with target,
   channel, expected response, and timeout behavior.

7. **Set LEARNING BOUNDARIES** — declare what can never change (IMMUTABLE), what
   needs human approval to change (MUTABLE_WITH_APPROVAL), and what can self-tune
   (AUTO_TUNABLE).

8. **Add FEEDBACK_RULEs** — if the workflow collects satisfaction signals from
   external parties, declare how positive patterns graduate into deterministic
   rules with thresholds, quality gates, and suspension conditions.

9. **Write SCENARIOS** — validate the workflow itself: happy path, rework loops,
   failure escalation, approval flows, rollback, and learning.

### The NALSD Test

Before finalizing, apply the litmus test: _could a human follow this OpSpec
step by step and produce the correct outcome?_ If any stage says "figure out the
right approach" or any gate says "use good judgment" without concrete criteria,
the OpSpec is incomplete. Fix the spec, not the coding agent.

---

Version: 0.6.0 | Author: Divyendu Deepak Singh | Date: 2026-04-07
