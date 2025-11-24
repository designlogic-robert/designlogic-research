# USE Routing Rules
How USE Dispatches Work to Micro-Engines and Domains

This document explains how the Universal Semantic Engine (USE) decides
**where each execution step goes next**: which micro-engine, when to call CE,
and when to hand off to Domain Engines.

Routing is fully deterministic and auditable.

---

# 1. Routing Overview

Routing answers three questions for every plan step from USR:

1. **What kind of work is this?**  
   (transform, expand, reduce, select, route, or domain-specific)

2. **Which micro-engine should handle it?**  
   (Transform, Expand, Reduce, Select, Route, or extension engine)

3. **What are the legal next hops?**  
   (another micro-engine, CE, Domain Engine, or completion)

Routing is implemented as a **rule table over structured plan steps**,  
not as ad-hoc if/else logic.

---

# 2. Routing Inputs

Each step arrives as a structured object:

```text
Step {
  id:                PlanStepID
  kind:              StepKind            # TRANSFORM | EXPAND | REDUCE | SELECT | ROUTE | DOMAIN_CALL
  token_family:      TokenFamilyID       # UST, Teleo, Trade, etc.
  posture:           CEPosture           # ANALYTIC | CREATIVE | CAUTIOUS | etc.
  invariants:        [InvariantID]
  domain_hint:       DomainID?           # optional (SynCE, FinCE, QLE, ...)
  budget:            PlanBudget          # steps, time, risk
  args:              SemanticArgs        # en
}
```
Routing uses only these structured fields plus current exec_state.

# 3. Core Routing Table

Base mapping from Step.kind to micro-engine:
```
Step.kind	Default Engine	Notes
TRANSFORM	TransformEngine	Identity-preserving semantic edits
EXPAND	ExpandEngine	Controlled semantic growth
REDUCE	ReduceEngine	Compression / summarization
SELECT	SelectEngine	Path choice, scoring, justification
ROUTE	RouteEngine	Meta-routing and cross-domain handoff
DOMAIN_CALL	RouteEngine	Delegates to Domain Engine registry
```
If an extension engine is registered for (token_family, kind), that engine
can override the default, subject to invariant checks.

---

# 4. Invariant Gate

Before routing executes, every step passes through an invariant gate:

Check that Step.invariants are compatible with:

token family

target engine

current CE posture

domain_hint (if present)

If any violation:

mark step as REJECTED

append violation to trace

halt or downgrade according to plan budget and risk policy

No step may be routed to an engine that cannot satisfy its invariants.

---

# 5. Posture-Aware Routing

CE posture influences where certain steps are allowed to go:

CAUTIOUS / ANALYTIC

strong preference for Reduce + Select engines

expansion limited or disallowed unless explicitly permitted

domain calls must justify risk in trace

EXPLORATORY / CREATIVE

greater use of Expand engine

more optional branches in Select

domain calls allowed for speculative reasoning (within budget)

DIAGNOSTIC / DEBUG

extra Reduce + Transform passes

emphasis on trace clarity over speed

Posture does not replace routing rules; it weights and filters them.

---

# 6. Domain Engine Routing

When Step.kind == DOMAIN_CALL, routing consults the Domain Engine registry:
```
DomainRegistry[DomainID] = {
  entrypoint:   EngineHandle,
  token_scope:  [TokenFamilyID],
  invariants:   [InvariantID],
  posture_mask: [AllowedPostures]
}
```

Routing checks:

Step.domain_hint exists in registry

Token family is allowed in that domain

CE posture is permitted

Invariants are compatible

If all checks pass:

control is handed off to the Domain Engine

a new domain trace segment is opened

USE waits for a structured result before continuing

If checks fail:

routing falls back to internal USE engines

a violation is recorded in the trace

---

# 7. Next-Hop Logic

After a micro-engine finishes, routing decides the next hop based on:

plan template from USR

remaining budget

engine result state (OK / PARTIAL / FAILURE)

completion criteria

Typical flows:

Transform → Reduce → Select → CE

Expand → Select → Domain Engine → Reduce

Transform → Domain Engine (fast path)

Reduce → Completion (answer already stable)

Every hop is appended to the execution trace.

---

# 8. Budget-Driven Routing

PlanBudget fields:

max_steps

max_time

risk_level

trace_granularity

Routing must:

decrement budget on each hop

halt if max_steps or max_time would be exceeded

avoid high-risk Domain Engines when risk_level is low

optionally skip non-critical passes when budget is tight

Budget decisions are always recorded in the trace.

---

# 9. Failure Handling

If a micro-engine fails or returns an invalid state:

Mark engine result as FAILURE

Add failure detail to trace

Route to Failure Handler path in plan:

attempt alternative engine or reduced mode

or halt with safe partial result

USE must never silently drop a failure.

---

# 10. Routing Invariants

Routing itself is governed by invariants:

Determinism: same plan + same state → same route

Traceability: every hop is logged with reasons

Safety First: invariant violations always override plan convenience

Extensibility: new engines can plug in via registry, not code forks

These invariants are what later allow USR and external auditors
to replay and verify USE behavior.

---

# 11. Summary

USE routing rules are the glue between:

USR’s high-level execution plans

USE micro-engines

CE postures

Domain Engines like SynCE, FinCE, and QLE

By treating routing as a first-class, deterministic system rather than
hidden control flow, USS can provide verifiable semantic execution
instead of opaque “AI magic.”