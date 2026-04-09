# OpSpec — Operational Specification Language

OpSpec is a general-purpose specification language for operational workflows. Any
process that has stages, decision gates, human interaction points, and the capacity
to learn and adapt can be expressed as an OpSpec.

A coding agent reads an OpSpec and produces the executor's program — a Temporal
workflow, a GitHub Actions pipeline, a Python module, shell scripts, or whatever
the target executor requires. The OpSpec is the specification; the produced program
is the executable. A human can also follow the same OpSpec manually, step by step.

```
OpSpec (specification)  →  Coding Agent  →  Executor Program (executable artifact)
                                                    ↓
                                            Executor runs it
                                            (Workflow engine, CI/CD pipeline, controller, shell runner)
```

OpSpec is an **independent specification language**. It can pair with NLSpec (a
specification standard for what to build) for software delivery workflows, but
OpSpec stands on its own. A claims processing OpSpec, a patient intake OpSpec,
or a supply chain OpSpec has no NLSpec dependency.

---

## What's in This Directory

| File | Purpose |
|------|---------|
| `OPSPEC-TEMPLATE.md` | **The OpSpec language definition and fill-in template.** 27 primitives across three categories (workflow, harness, coordination), section structure, executor requirements, binding types, learning governance, scenario conventions, NLSpec mapping, and worked examples. Start here to write an OpSpec. |

---

## Quick Start

1. Read `OPSPEC-TEMPLATE.md` — it is both the language reference and a fill-in template
2. Copy the template and fill in your workflow's stages, gates, bindings, and learning rules
3. Hand the completed OpSpec to a coding agent to produce the executor program

---

## Primitives

OpSpec has 27 primitives in three categories.

### Workflow Primitives

The backbone of any OpSpec. Referenced by stages as dependencies, executed in sequence.

**STAGE** — A phase of work with entry conditions, concrete operations, and exit conditions.

**GATE** — A three-band decision point (PASS / REWORK / FAIL) between stages. Every GATE has concrete criteria — no "use good judgment."

**BINDING** — An artifact template (prompt, rule, tool config) that the coding agent produces as part of the executor program.

**LEARNING RULE** — Governed self-modification: the workflow observes patterns, proposes changes to itself, and submits them for approval within declared boundaries.

**APPROVAL** — A human interaction point where the executor pauses and waits. Every APPROVAL has a channel, SLA, and escalation path.

**INTERACTION** — A customer-facing touchpoint where the workflow communicates with an external party (customer, patient, applicant). Unlike APPROVAL (internal review), INTERACTION represents outbound communication that may await a response or collect feedback.

**FEEDBACK_RULE** — Satisfaction-driven graduation: the workflow observes customer feedback signals (ratings, reopens, escalation requests), accumulates positive patterns, and graduates them into deterministic rules that bypass LLM stages.

**TOPOLOGY** — Infrastructure layout: provider, region, services, bindings. The operational surface the OpSpec operates on.

**PIPELINE** — Execution structure with stage ordering and dependencies.

**RULE** — A deterministic condition-action pair with lifecycle (draft/active/suspended/retired), versioning, priority-based conflict resolution, and CEL support for complex matching.

**RECORD** — Structured data referenced by stages and bindings. A data model or schema.

**CONDUIT** — An adapter that converts OpSpec instructions into concrete config files for an operational surface (Kubernetes, Cloudflare, AWS, etc.). Has a four-phase lifecycle: plan, generate, validate, drift. Handles version negotiation and schema validation.

### Harness Primitives

Consumed by the executor's runtime, not by individual workflow stages. These configure runtime behavior: monitoring, scheduling, deployment strategy, incident response.

**RUNBOOK** — Human-readable operational procedures. Reference material for incident response, consulted by both humans and agents.

**LIFECYCLE** — A state machine for deployment or operational lifecycle. Defines valid state transitions.

**CONFIG** — Runtime configuration consumed by the executor. Environment variables, knobs, and tunable parameters.

**METRIC** — A monitoring condition with source, threshold, and action. How OpSpecs express health checks, SLOs, and alerting.

**SCHEDULE** — A cron-triggered operation. Time-based execution.

**STRATEGY** — Deployment strategy (canary, blue-green, rolling, immediate) with rollout parameters.

**ROLLBACK** — Recovery trigger and actions. What fires a rollback and what the rollback does.

**PROMOTE** — Environment promotion with gates. How artifacts move from staging to production.

**ESCALATION** — Escalation policy with severity tiers, channels, and timeouts.

**NOTIFY** — Notification template with channel and runtime variables.

**GRADUATED** — A learned pattern that has been graduated into a deterministic rule. Bypasses reasoning.

**REMEDIATION** — Autonomous remediation with scope, validation, and fallback.

### Coordination Primitives

Used for multi-agent communication in deployments where multiple executors collaborate.

**CONTRACT** — A capability declaration for multi-agent coordination. What an executor can do, what surfaces it manages, what constraints apply.

**FENCE** — CAS-based resource fencing for concurrent executor safety. Multiple readers, exclusive writer. TTL-based expiry prevents dead executors from holding fences indefinitely. No central lock manager.

**RECONCILE** — Leaderless distributed reconciliation via epoch-based CAS. Every executor independently observes shared state, proposes mutations, and applies via compare-and-swap. Includes distributed takeover for reclaiming orphaned tasks from dead executors.

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

9. **Add Operational Primitives** — declare TOPOLOGYs for infrastructure layout,
   METRICs for monitoring, SCHEDULEs for cron operations, STRATEGYs for deployment
   rollout, ESCALATIONs for incident routing, CONDUITs for config generation,
   CONTRACTs for multi-agent coordination, FENCEs for resource safety, and
   RECONCILEs for distributed consensus.

10. **Write SCENARIOS** — validate the workflow itself: happy path, rework loops,
    failure escalation, approval flows, rollback, and learning.

### The NALSD Test

Before finalizing, apply the litmus test: _could a human follow this OpSpec
step by step and produce the correct outcome?_ If any stage says "figure out the
right approach" or any gate says "use good judgment" without concrete criteria,
the OpSpec is incomplete. Fix the spec, not the coding agent.

---

Version: 0.9.0 | Author: Divyendu Deepak Singh | Date: 2026-04-08
