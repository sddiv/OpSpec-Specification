# {WORKFLOW_NAME} — Operational Specification

> **Version:** 0.1.0
> **Author:** {AUTHOR}
> **Date:** {DATE}
> **Status:** {Draft | Review | Approved | Active | Retired}

---

## How to Use This Spec

This document is a **complete operational specification** — the blueprint from which
a coding agent produces the executor's program. The OpSpec is not the program
itself. A coding agent reads this OpSpec and produces the actual executable artifact:
a Rust crate, a Python program, a prompt, a Temporal workflow definition,
or whatever implementation the target executor requires.

Every STAGE, GATE, and BINDING is precise enough that a coding agent can translate it
into executable code. Every DECISION has concrete criteria. If a coding agent must
guess "what should happen here?", the OpSpec is incomplete — fix the OpSpec.

A human can also follow this spec manually, step by step. That is the litmus test:
if a human can't follow it, a coding agent can't produce a correct program from it.

**For the coding agent:** Read every STAGE in order. At each GATE, translate the
DECISION criteria into executable logic. Generate executor artifacts from BINDINGs.
Implement LEARNING RULE boundaries as governance constraints. If a GATE fails,
implement the declared failure path — do not invent one.

**For the executor:** Load this OpSpec, verify all EXECUTOR REQUIREMENTS are met,
and execute the STAGEs and GATEs in order. At each APPROVAL, pause and notify. At
each LEARNING RULE, monitor for observations and propose changes within boundaries.

### What OpSpec Is

OpSpec is a **general-purpose operational specification language**. Any workflow
that has stages, decision gates, human interaction points, and the capacity to
learn and adapt can be expressed as an OpSpec. A coding agent reads an OpSpec
and produces the executor's program — the actual executable artifact. When the
executor is a controller, it produces a controller program. When the executor is
a CI/CD pipeline, it produces pipeline configuration. When the executor is a shell
runner, it produces scripts.

```
OpSpec (specification)  →  Coding Agent  →  Executor Program (executable artifact)
                                                    ↓
                                            Executor runs it
                                            (Workflow engine, CI/CD pipeline, shell runner, etc.)
```

This is symmetric with NLSpec (a published declarative specification language):
- **NLSpec** → coding agent → **application code** (Rust, Python, etc.)
- **OpSpec** → coding agent → **executor program** (workflow, pipeline config, scripts, etc.)

NLSpec and OpSpec are **independent specifications**. Each stands on its own. They compose when
a business requirement calls for it — a software delivery pipeline uses an NLSpec as its
source spec, but a claims processing pipeline has no NLSpec dependency.

**OpSpec applications include but are not limited to:**
- Software delivery pipelines (code → build → test → deploy → monitor)
- Insurance claims processing (intake → verify → assess → adjudicate → pay)
- Loan origination (application → underwrite → approve → fund → service)
- Patient intake and triage (register → assess → route → treat → follow-up)
- Supply chain orchestration (order → source → manufacture → ship → deliver)
- Content moderation (receive → classify → review → act → appeal)
- Customer onboarding (sign-up → verify → provision → train → activate)

OpSpec can pair with NLSpec for software delivery workflows: NLSpec defines **what** to
build, OpSpec defines **how** to deliver and operate it. But OpSpec stands on its own —
a claims processing OpSpec has no NLSpec dependency.

### Why OpSpec Exists (Why NLSpec Can't Do This)

The fundamental difference is **scope**: NLSpec works on one artifact at a time.
OpSpec produces many artifacts across many stages in a single pipeline run.

An NLSpec says "here is what this one system does" — one spec, one binary, one
deployable. A software delivery OpSpec might produce a Rust crate from one NLSpec,
a Docker image, a Kubernetes manifest, a database migration, and a monitoring
config — all coordinated with decision gates between them. A claims processing
OpSpec might produce an intake form, a fraud scoring model, an adjudication
workflow, a payment instruction, and an audit report. OpSpec is the thing that
knows the order, the gates between stages, and what to do when stage 3 fails.

This is also why OpSpec needs LEARNING RULES and NLSpec doesn't. A pipeline that
produces many artifacts across many runs accumulates patterns about what works
and what doesn't. NLSpec describes a static system; OpSpec describes a living
process that improves over time.

Beyond scope, NLSpec is **declarative** — it describes *what* a system does and
trusts the coding agent to produce correct code. This works because each NLSpec
produces one artifact. A single system has a bounded surface area.

OpSpec manages many artifacts in a single pipeline — and those artifacts are
products going to production. A claims pipeline produces an intake form, a fraud
model, an adjudication workflow, a payment instruction, and an audit report. A
software delivery pipeline produces a binary, a container image, a manifest, a
migration, and a monitoring config. Getting any one wrong is a defect. Getting the
coordination between them wrong is an outage.

OpSpec is **procedurally explicit** because precision is the point. Every STAGE
spells out concrete operations. Every GATE spells out exact three-band criteria.
Every BINDING spells out the template with variables. The coding agent has enough
procedural detail to produce each artifact correctly and coordinate them precisely.

```
                    NLSpec                          OpSpec
Scope:              One artifact                    Many artifacts, one pipeline
Style:              Declarative                     Procedural
Output:             Application code                Executor program (pipeline, controller, scripts)
Learning:           Static (system doesn't evolve)  Adaptive (LEARNING RULEs accumulate patterns)
Why it's procedural: Single artifact, bounded scope  Multiple artifacts, precision required
```

### What OpSpec Is Not

- **Not a program.** OpSpec is a specification, not an executable. English is not
  a runtime. A coding agent reads the OpSpec and produces the actual program —
  a Rust crate, a Python module, a prompt, a pipeline config — whatever
  the target executor requires.
- **Not a pipeline config.** Jenkins, Temporal, Airflow are execution engines.
  OpSpec is the specification from which a coding agent *produces* pipeline
  configurations, controller programs, or agent prompts.
- **Not a runbook.** Runbooks describe how humans react to incidents. OpSpec
  describes workflow behavior completely enough that a coding agent can produce
  a program that operates the workflow proactively.
- **Not an NLSpec.** OpSpec is a separate specification language with its
  own primitives, designed for operational workflows. It can reference an NLSpec
  when used for software delivery, but does not require it.

### Executor Compatibility

An OpSpec declares the **minimum executor capabilities** the produced program
targets. Executors are diverse:

**Executor types:**
- **Workflow Engine** — Temporal, Airflow, Step Functions. Has stages and decision points with built-in orchestration.
- **CI/CD Pipeline** — GitHub Actions, Jenkins, GitLab CI. Has stages, gates, and artifact management.
- **Controller** — Any programmable runtime that executes the workflow. Custom logic, full control.
- **Shell Runner** — Sequential command execution. Minimal. No complex decision logic.
- **Custom** — Any executor that implements the required capabilities.

Not every executor program can run on every executor, just as not every
Android app can run on every Android phone.

---

## OpSpec Primitives

OpSpec has its own primitives designed for operational workflows:

### STAGE — A Phase of Work

A STAGE is a discrete phase in the operational workflow. Stages execute in
declared order. Each stage has a clear entry condition, a set of operations,
and an exit condition.

```
STAGE {number}: {name}
  ENTRY:
  - {condition that must be true to enter this stage}

  OPERATIONS:
  1. {concrete action — command, API call, agent prompt, or human task}
  2. {concrete action}
  3. {concrete action}

  EXIT:
  - {condition that must be true to leave this stage}

  ON_FAILURE:
  - {what happens if EXIT conditions are not met}

  PRODUCES:
  - {artifact or state change this stage creates}

  DURATION:
  - expected: {typical time}
  - timeout:  {maximum time before escalation}
```

### GATE — A Decision Point

A GATE sits between stages. It evaluates conditions and routes the workflow.
Gates are decision points evaluated by whatever system runs them.

```
GATE {number}: {name}
  EVALUATES: {what is being assessed — test results, metrics, human feedback, etc.}

  DECISION:
    PASS:
      criteria  : {concrete, measurable condition}
      action    : {PROCEED to STAGE N}

    REWORK:
      criteria  : {concrete, measurable condition — the "in-between" band}
      action    : {RETURN to STAGE N with {context}}
      max_cycles: {integer — maximum rework loops before escalation}

    FAIL:
      criteria  : {concrete, measurable condition}
      action    : {ESCALATE to {human_role} | ABORT | ROLLBACK to STAGE N}

  TIMEOUT:   {duration before automatic escalation}
  AUDIT:     {what gets logged for this gate decision}
```

**Three-band decision model:** Every GATE has three outcomes — PASS, REWORK,
FAIL. This is not binary pass/fail. The middle band (REWORK) is where most
real-world iteration happens. The executor sends work back with context
about what needs fixing, and the upstream stage tries again.

### BINDING — Agent Artifact Generation

A BINDING declares what agent artifacts a stage or gate requires. This is where
OpSpec is *generative* — it doesn't just describe steps, it specifies the prompts,
rules, MCP bindings, and configurations that the coding agent must produce as
part of the executor program.

```
BINDING {name}:
  type        : {PROMPT | RULE | MCP_TOOL | CONFIG}
  stage       : {STAGE number this binding is used in}
  generates   : {what this binding produces}
  template    : |
    {the actual template content — prompt text, rule definition,
     MCP tool configuration, etc.}
  variables   : [{variable names that get filled in at runtime}]
```

**BINDING types:**
- `PROMPT` — A prompt template that the produced program sends to an agent or reasoning service.
  Variables are filled with runtime context.
- `RULE` — A deterministic rule or check the executor enforces during execution.
  Defines constraints, thresholds, or validation logic.
- `MCP_TOOL` — An MCP tool binding used during this workflow.
  Declares the tool, its parameters, and when to invoke it.
- `CONFIG` — A configuration artifact (pipeline config, form template,
  notification template, routing rules, etc.).

### LEARNING RULE — Adaptive Behavior Within Governance

A LEARNING RULE defines what the OpSpec can learn and how it proposes changes
to itself. This is what makes OpSpec adaptive — the workflow evolves based on
outcomes, but only within declared boundaries and with human approval.

```
LEARNING RULE {name}:
  observes    : {what the running program monitors — metrics, patterns, outcomes}
  learns      : {what conclusion the program can draw}
  proposes    : {what OpSpec change the program can suggest}
  boundary    : {what CANNOT change — hard limits on self-modification}
  approval    : {HUMAN_REQUIRED | AUTO_APPROVE_IF {condition}}
  evidence    : {minimum evidence required before proposing — e.g., "3 consecutive occurrences"}
```

**Self-modification loop:**
1. Controller observes pattern during execution (per LEARNING RULE)
2. Controller accumulates evidence (per `evidence` threshold)
3. Controller proposes OpSpec amendment (per `proposes` template)
4. Amendment goes through approval (per `approval` policy)
5. If approved, OpSpec updates. If rejected, learning rule records the rejection.
6. Controller's `boundary` field is NEVER modifiable by the learning process itself.

### APPROVAL — Human Interaction Point

An APPROVAL declares where humans enter the workflow. The executor
operates autonomously until it hits an APPROVAL, then pauses, notifies,
and waits.

```
APPROVAL {name}:
  required_for : {STAGE or GATE number}
  approvers    : {role or team}
  channel      : {how to notify — Slack, email, dashboard, etc.}
  min_approvals: {integer}
  sla          : {expected response time}
  escalation   : {what happens if SLA is missed}
  bypass       : {conditions for emergency bypass, or "NEVER"}
  audit        : true  -- always logged
```

---

## Section Declaration

OpSpecs declare their sections at the top. When present, the SECTIONS block
is the contract — it tells the controller what to expect and enables validation.

```
SECTIONS:
  Abstract             — what this workflow does, its inputs and outputs
  ExecutorRequirements — minimum executor capabilities, hardware, integrations
  Workflow             — STAGEs and GATEs in execution order
  Bindings             — agent artifact templates (prompts, rules, MCP tools, configs)
  Approvals            — human interaction points
  Learning             — adaptive behavior rules and governance
  Observability        — what the executor monitors and reports
  Rollback             — recovery procedures for each stage
  Scenarios            — validation scenarios for the workflow itself
  Boundaries           — what this OpSpec does NOT cover
  Contracts            — what this OpSpec exports to and expects from other specs
```

---

## Abstract

**One paragraph.** What this workflow does, in plain English. A human reading only
this paragraph should understand the operational scope and purpose.

```
{WORKFLOW_NAME} is the operational specification for {what this workflow does}.

DOMAIN:  {domain — e.g., "software delivery", "claims processing", "patient intake"}

It takes {input — e.g., "a validated spec and manifest", "a submitted claim form",
"a patient registration"} and produces {output — e.g., "a deployed production system",
"an adjudicated claim with payment", "a triaged patient with care plan"}.

The primary human interaction points are: {list approval gates}.
```

**For software delivery workflows** that operate on a source specification, add:
```
SOURCE SPEC: {source-spec.md}        -- optional, for delivery workflows
MANIFEST:    {manifest.toml}         -- optional, for delivery workflows
```

**For other operational workflows**, this section is simply:
```
{Plain English description of what this workflow does, its inputs/outputs, and executor requirements}
```

---

## ExecutorRequirements

Declares the minimum executor capabilities needed to run this OpSpec.
This is the Android `<uses-feature>` equivalent.

```
EXECUTOR REQUIREMENTS:
  executor_type     : {WORKFLOW_ENGINE | CICD_PIPELINE | CONTROLLER | SHELL_RUNNER | CUSTOM}

  hardware:
    network         : {internet_required | air_gap_ok | local_only}
    persistent_state: {true | false}

  integrations:
    - {MCP server, API, tool, or external service}

  COMPATIBILITY CHECK:
    command         : {how the executor verifies requirements}
    fail_action     : {ABORT | WARN | DEGRADE}
```

---

## Workflow

The core of the OpSpec. STAGEs and GATEs in execution order. This section
is what a human can follow manually, or what a coding agent translates into
the executor program's execution logic.

### Workflow Conventions

- STAGEs are numbered sequentially: STAGE 1, STAGE 2, etc.
- GATEs sit between stages: GATE 1 (after STAGE 1), GATE 2 (after STAGE 2), etc.
- Rework loops go backward: GATE 2 → REWORK → return to STAGE 2 (or earlier).
- The workflow forms a DAG (directed acyclic graph) with rework being bounded loops,
  not unbounded cycles. Every rework loop has a `max_cycles` limit.
- Parallel stages are declared with `PARALLEL:` blocks.

### Workflow Template

```
STAGE 1: {name}
  ENTRY:
  - {precondition}

  OPERATIONS:
  1. {concrete action}
  2. {concrete action}
  3. {concrete action}

  EXIT:
  - {postcondition}

  ON_FAILURE:
  - {failure action}

  PRODUCES:
  - {artifact}

  DURATION:
  - expected: {time}
  - timeout:  {time}

---

GATE 1: {name}
  EVALUATES: {what}

  DECISION:
    PASS:
      criteria  : {measurable}
      action    : PROCEED to STAGE 2

    REWORK:
      criteria  : {measurable}
      action    : RETURN to STAGE 1 with {context}
      max_cycles: {N}

    FAIL:
      criteria  : {measurable}
      action    : ESCALATE to {role}

---

STAGE 2: {name}
  ...
```

### Parallel Stages

When stages can execute concurrently:

```
PARALLEL:
  STAGE 3a: {name}
    ...
  STAGE 3b: {name}
    ...
  JOIN: ALL_COMPLETE | ANY_COMPLETE | QUORUM({n})
```

### Conditional Stages

When a stage only executes under certain conditions:

```
STAGE 5: {name}
  CONDITION: {when this stage applies — e.g., "only if GATE 4 produced artifacts"}
  ...
```

---

## Bindings

Agent artifact templates that the executor generates and uses during
workflow execution. This is what makes OpSpec generative — the specification
produces the executor's runtime behavior.

### Binding Conventions

```
BINDING {name}:
  type        : PROMPT | RULE | MCP_TOOL | CONFIG
  stage       : STAGE {N}
  generates   : {description of what this produces}

  template    : |
    {The actual artifact template. For PROMPTs, this is the prompt text
     with ${variables} for runtime substitution. For RULEs,
     this is the rule definition. For CONFIGs, this is the config
     template.}

  variables:
    - {variable_name} : {source — where the value comes from at runtime}
```

### Standard Binding Types

**PROMPT bindings** — templates for agent instructions:
```
-- Example: Software delivery workflow
BINDING code_review_prompt:
  type        : PROMPT
  stage       : STAGE 3
  generates   : Prompt for agent to review and fix test failures

  template    : |
    You are reviewing code for: ${project_name}.
    The following tests failed:
    ${test_failures}
    Fix the code to pass these tests. Do not modify the tests.

  variables:
    - project_name  : from Abstract
    - test_failures : from STAGE 3 test execution output

-- Example: Claims processing workflow
BINDING claim_assessment_prompt:
  type        : PROMPT
  stage       : STAGE 3
  generates   : Prompt for agent to assess claim validity

  template    : |
    Assess the following insurance claim against policy ${policy_id}.
    Claim details: ${claim_summary}
    Policy coverage: ${coverage_terms}
    Prior claims history: ${claims_history}
    Determine: covered (PASS), needs investigation (REWORK), or denied (FAIL).

  variables:
    - policy_id      : from STAGE 1 output (policy lookup)
    - claim_summary  : from STAGE 2 output (intake)
    - coverage_terms : from MCP_TOOL policy_database query
    - claims_history : from MCP_TOOL claims_database query
```

**RULE bindings** — deterministic rules the executor enforces:
```
-- Example: Software delivery — shipping cadence
BINDING ship_readiness_rule:
  type        : RULE
  stage       : GATE 5
  generates   : Rule that enforces shipping readiness criteria

  template    : |
    IF days_since_last_ship > ${max_days} THEN bias = SHIP
    IF critical_bugs > 0 THEN bias = HOLD
    IF test_coverage < ${min_coverage} THEN decision = REWORK

  variables:
    - max_days      : from OpSpec Config section
    - min_coverage  : from OpSpec Config section

-- Example: Claims processing — fraud detection
BINDING fraud_detection_rule:
  type        : RULE
  stage       : GATE 2
  generates   : Rule that flags potentially fraudulent claims

  template    : |
    IF claim_amount > ${threshold} AND claims_in_90d > ${max_frequency} THEN flag = FRAUD_REVIEW
    IF provider_id IN ${watchlist} THEN flag = FRAUD_REVIEW
    IF claim_date < policy_start_date THEN flag = DENY

  variables:
    - threshold     : from OpSpec Config (e.g., $10,000)
    - max_frequency : from OpSpec Config (e.g., 3)
    - watchlist     : from MCP_TOOL fraud_database query
```

**MCP_TOOL bindings** — tool integrations:
```
BINDING case_management_tool:
  type        : MCP_TOOL
  stage       : STAGE 4
  generates   : MCP tool binding for creating/updating cases

  template    : |
    MCP_SERVER: ${case_system}
    TOOL: update_case
    PARAMS:
      case_id   : ${case_id}
      status    : ${new_status}
      notes     : ${decision_notes}
      assignee  : ${next_handler}

  variables:
    - case_system    : from ExecutorRequirements.integrations
    - case_id        : from STAGE 1 output
    - new_status     : from GATE decision
    - decision_notes : generated by agent
    - next_handler   : from routing rules
```

**CONFIG bindings** — generated configurations:
```
BINDING notification_template:
  type        : CONFIG
  stage       : STAGE 5
  generates   : Customer notification email/SMS template

  template    : |
    TO: ${customer_contact}
    CHANNEL: ${preferred_channel}
    TEMPLATE: ${outcome_template}
    VARS:
      name          : ${customer_name}
      case_number   : ${case_id}
      decision      : ${decision}
      next_steps    : ${next_steps}

  variables:
    - customer_contact   : from case record
    - preferred_channel  : from customer preferences
    - outcome_template   : from decision type (approved/denied/pending)
    - customer_name      : from case record
    - case_id            : from STAGE 1 output
    - decision           : from GATE output
    - next_steps         : generated by agent
```

---

## Approvals

Human interaction points. The executor program operates autonomously between
approvals. At each approval, it pauses, notifies, and waits.

```
APPROVAL {name}:
  required_for : STAGE {N} | GATE {N}
  approvers    : {role — e.g., "engineering-lead", "product-owner", "security-team"}
  channel      : {notification method — e.g., "slack:#deployments", "email:team@company.com"}
  min_approvals: {integer}
  sla          : {expected response time — e.g., "4h", "1 business day"}
  escalation:
    after       : {duration}
    to          : {escalation target}
    auto_action : {what happens if escalation also times out — e.g., "ABORT", "PROCEED_WITH_WARNING"}
  bypass:
    condition   : {when bypass is allowed — e.g., "active incident with severity >= P1"}
    requires    : {what's needed for bypass — e.g., "incident_id"}
    post_approval: {whether retroactive approval is needed — true/false}
    post_sla    : {time to obtain retroactive approval}
  audit         : true  -- always
```

### Approval Flow

```
Executor reaches APPROVAL point
  → Executor pauses workflow
  → Executor sends notification via {channel}
  → Executor includes: what was done, what needs approval, relevant artifacts
  → Executor waits for {min_approvals} approvals
  → If SLA exceeded: Executor escalates
  → If approved: Executor proceeds to next STAGE
  → If rejected: Executor follows rejection path (defined per approval)
  → All decisions logged to audit trail
```

---

## Learning

LEARNING RULEs that allow the OpSpec to evolve based on operational outcomes.
This is the adaptive layer — bounded, governed, and auditable.

### Learning Conventions

```
LEARNING RULE {name}:
  observes    : {what the executor watches during execution}
  window      : {observation window — e.g., "last 10 executions", "30 days"}
  learns      : {what pattern or insight the executor detects}
  proposes    : {what OpSpec change to suggest}
  boundary    : {HARD — what the executor CANNOT modify, even with approval}
  evidence    : {minimum evidence before proposing — e.g., "3 occurrences in window"}
  approval    : HUMAN_REQUIRED | AUTO_APPROVE_IF {condition}
  rollback    : {how to undo the learned change if it causes problems}
```

### Learning Boundaries (hard limits)

Every OpSpec MUST declare what learning CANNOT modify. These boundaries are
enforced by the executor and are themselves immutable by the learning process.

```
LEARNING BOUNDARIES:
  IMMUTABLE:
    - {what can never change — e.g., "APPROVAL gates cannot be removed"}
    - {what can never change — e.g., "FAIL criteria in GATEs cannot be relaxed"}
    - {what can never change — e.g., "security checks cannot be skipped"}
    - {what can never change — e.g., "human approval for critical actions"}
  MUTABLE_WITH_APPROVAL:
    - {what can change with human approval — e.g., "REWORK thresholds"}
    - {what can change — e.g., "timeout durations"}
    - {what can change — e.g., "PASS criteria in non-critical GATEs"}
  AUTO_TUNABLE:
    - {what the executor can adjust autonomously — e.g., "parallelism levels"}
    - {what can auto-tune — e.g., "retry counts within bounds"}
```

### Self-Modification Audit Trail

Every learned change is recorded:

```
LEARNING LOG ENTRY:
  rule        : {LEARNING RULE name}
  timestamp   : {when}
  evidence    : {observations that triggered the proposal}
  proposal    : {what change was proposed}
  approval    : {APPROVED | REJECTED | AUTO_APPROVED}
  approved_by : {human name or "AUTO"}
  applied     : {timestamp when change took effect, or "N/A" if rejected}
  reverted    : {timestamp if rolled back, or "N/A"}
```

---

## Observability

What the executor program monitors and reports during workflow execution.
Separate from the *application's* observability — this is the *workflow's* observability.

```
WORKFLOW METRICS:
  - {metric_name}: {what it measures}
    type    : {counter | gauge | histogram | summary}
    labels  : [{label names}]
    alert   : {condition that triggers alert, or "none"}

WORKFLOW LOGS:
  format    : {structured JSON | plaintext}
  level     : {what gets logged at each level}
  retention : {how long logs are kept}
  destination: {where logs go}

WORKFLOW DASHBOARD:
  provider  : {Grafana | Datadog | custom | etc.}
  panels    : [{what each panel shows}]
```

### Standard Workflow Metrics (every OpSpec should include):

```
METRIC workflow_stage_duration:
  measures : Time spent in each STAGE
  type     : histogram
  labels   : [stage_name, outcome]
  alert    : "stage_duration > 2x expected duration → WARN"

METRIC workflow_gate_decisions:
  measures : GATE outcomes (PASS/REWORK/FAIL counts)
  type     : counter
  labels   : [gate_name, decision]
  alert    : "rework_rate > 50% over 5 executions → WARN"

METRIC workflow_rework_cycles:
  measures : Number of rework iterations per gate
  type     : histogram
  labels   : [gate_name]
  alert    : "rework_cycles approaching max_cycles → WARN"

METRIC workflow_human_wait_time:
  measures : Time waiting for human approval
  type     : histogram
  labels   : [approval_name]
  alert    : "wait_time > SLA → escalation"

METRIC workflow_learning_proposals:
  measures : Learning rule activations
  type     : counter
  labels   : [rule_name, outcome]
  alert    : none
```

---

## Rollback

Recovery procedures for each stage. Rollback is not a hope —
it is a tested, concrete procedure that the executor can execute.

```
ROLLBACK {stage_name}:
  trigger     : {what triggers rollback — automatic condition or manual}
  procedure:
    1. {concrete step}
    2. {concrete step}
    3. {concrete step}
  verify      : {how to confirm rollback succeeded}
  duration    : {maximum acceptable rollback time}
  data_impact : {what happens to data — "no data loss", "last N minutes lost", etc.}
  notification: {who gets notified}
```

### Rollback Principles

1. **Every stage that changes external state must have a rollback procedure.**
   Stages that only read or analyze don't need rollback.
2. **Rollback is tested.** Each rollback procedure has a SCENARIO in the Scenarios
   section that validates it works.
3. **Rollback is bounded.** Maximum rollback duration is declared. If exceeded,
   escalation occurs.
4. **Data impact is explicit.** The executor (and humans) know exactly what
   data loss to expect from a rollback.

---

## Scenarios

Validation scenarios for the **workflow itself** — not the domain application.
(If the OpSpec delivers software, the NLSpec covers application scenarios. If the OpSpec
processes claims, the policy document covers claim scenarios.) These scenarios
validate that the OpSpec's stages, gates, bindings, and learning rules work correctly.

### Conventions

```
SCENARIO {number}: {name}
  GIVEN:
  - {workflow state — e.g., "STAGE 2 complete, test coverage at 75%"}

  WHEN:
  - {workflow event — e.g., "GATE 2 evaluates test results"}

  THEN:
  - {expected workflow behavior — e.g., "GATE 2 returns REWORK, sends to STAGE 2"}
  - {expected notification — e.g., "executor receives rework context"}

  VALIDATES: {which OpSpec element — STAGE, GATE, BINDING, LEARNING RULE, APPROVAL}
```

### Standard Scenario Categories

```
Scenarios.1 Happy Path
  — Full workflow from start to completion, all gates PASS

Scenarios.2 Rework Loops
  — Each GATE's REWORK path exercised, verify bounded iteration

Scenarios.3 Failure Escalation
  — Each GATE's FAIL path exercised, verify human notification

Scenarios.4 Approval Flows
  — Each APPROVAL exercised: approve, reject, timeout, bypass

Scenarios.5 Rollback
  — Each ROLLBACK procedure executed and verified

Scenarios.6 Learning
  — Each LEARNING RULE triggered, proposal generated, approval flow exercised

Scenarios.7 Executor Compatibility
  — OpSpec loaded on incompatible executor, verify graceful rejection
```

---

## Boundaries

**What this OpSpec does NOT cover.**

```
### This OpSpec Does:
- {what this workflow covers}

### This OpSpec Does NOT:
- {non-goal} — {what handles it instead}

### Future Extensions:
- {capability that may be added later}
```

---

## Contracts

What this OpSpec exports and expects. OpSpecs can compose — one workflow
OpSpec might expect artifacts from another OpSpec (e.g., a build workflow
feeds into a delivery workflow, or an intake workflow feeds into an
adjudication workflow).

### Contracts.1 Exports

```
EXPORT {name}:
  type        : ARTIFACT | STATE | METRIC | EVENT
  description : {what is exported}
  consumers   : {who uses this — other OpSpecs, humans, external systems}
```

### Contracts.2 Expects

```
EXPECTS {name}:
  from        : {source — another OpSpec, external system, human input, database}
  type        : ARTIFACT | STATE | CREDENTIAL | CONFIG
  description : {what is needed}
  required    : true | false
  fallback    : {default if not provided}
```

### Contracts.3 Source Specification Integration (Optional)

When an OpSpec operates on a source specification (e.g., an NLSpec for software
delivery, a policy document for claims processing, a curriculum standard for
education), declare the mapping here. This section is OPTIONAL — not all
OpSpecs have a source specification.

```
SOURCE SPEC: {spec-file}               -- optional
MANIFEST:    {manifest-file}            -- optional

SOURCE MAPPING:
  STAGE {N} reads  : [SEC:{section}]    -- which source spec sections this stage needs
  GATE {N}  uses   : [SEC:{section}]    -- source sections used for gate validation
  BINDING {name}   : [SEC:{section}]    -- which source section the binding references
```

**Examples:**
- Software delivery: `SOURCE SPEC: delivery-spec.md` (NLSpec)
- Claims processing: `SOURCE SPEC: auto-insurance-policy-v3.pdf` (policy doc)
- Patient intake: `SOURCE SPEC: hospital-triage-protocol.md` (clinical protocol)
- Content moderation: `SOURCE SPEC: community-guidelines-v2.md` (policy doc)

---

## Config

Operational parameters that tune the workflow without changing its structure.
These are the "knobs" a human adjusts.

```
CONFIG {group}:
  {key}:
    type        : {Type}
    default     : {value}
    description : {what it controls}
    validation  : {constraints}
    mutable_by  : {HUMAN_ONLY | LEARNING_RULE | AUTO_TUNE}
```

### Standard Config Groups

```
CONFIG cadence:
  cycle_interval:
    type        : Duration
    default     : 2d
    description : Target cycle time (time between completed workflow runs)
    validation  : >= 1h
    mutable_by  : HUMAN_ONLY

  rework_max_cycles:
    type        : u32
    default     : 3
    description : Maximum rework iterations before escalation
    validation  : >= 1, <= 10
    mutable_by  : LEARNING_RULE

CONFIG thresholds:
  gate_pass_minimum:
    type        : f64
    default     : 0.60
    description : Below this, escalate to human (FAIL band)
    validation  : 0.0 - 1.0
    mutable_by  : HUMAN_ONLY

  gate_pass_target:
    type        : f64
    default     : 0.90
    description : Above this, proceed (PASS band). Between minimum and target = REWORK.
    validation  : > gate_pass_minimum, <= 1.0
    mutable_by  : LEARNING_RULE

CONFIG timeouts:
  stage_default_timeout:
    type        : Duration
    default     : 30m
    description : Default maximum time for any stage
    validation  : >= 5m
    mutable_by  : AUTO_TUNE

  approval_sla:
    type        : Duration
    default     : 4h
    description : Default SLA for human approvals
    validation  : >= 1h
    mutable_by  : HUMAN_ONLY
```

---

## Appendix A: OpSpec Primitives at a Glance

| Primitive | Purpose | Analogy |
|-----------|---------|---------|
| **STAGE** | A phase of work with inputs, operations, outputs | A step in a process flowchart |
| **GATE** | A decision point with three-band outcome (PASS/REWORK/FAIL) | A quality checkpoint |
| **BINDING** | Generates agent artifacts (prompts, rules, configs, tools) | A factory that produces the executor's runtime behavior |
| **LEARNING RULE** | Governed self-modification based on observed patterns | A feedback loop with guardrails |
| **APPROVAL** | Human interaction point — the executor pauses and waits | A sign-off gate in any approval chain |

### NLSpec Primitive Mapping (for software delivery workflows)

When OpSpec is used alongside an NLSpec for software delivery, the primitives map as follows:

| NLSpec Primitive | OpSpec Primitive | Parallel |
|-----------------|-----------------|----------|
| RECORD | — | OpSpec doesn't define data structures — it references the NLSpec's |
| FUNCTION | STAGE | A phase of work with inputs, operations, outputs |
| SCENARIO | SCENARIO | Validation of expected behavior |
| — | GATE | Decision points (no NLSpec equivalent) |
| — | BINDING | Agent artifact generation (no NLSpec equivalent) |
| — | LEARNING RULE | Self-modification (no NLSpec equivalent) |
| — | APPROVAL | Human interaction (no NLSpec equivalent) |
| DEPENDENCY | EXPECTS | External requirements |
| EXPORT | EXPORT | What this spec provides to others |
| Config | Config | Tunable parameters |

## Appendix B: Revision History

```
| Version | Date       | Author | Changes                    |
|---------|------------|--------|----------------------------|
| 0.1.0   | {date}     | {name} | Initial draft              |
```
