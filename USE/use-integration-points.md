# USE Integration Points
Execution Layer Interfaces Across UST → USR → CE → Domain Engines

The Universal Semantic Engine (USE) sits at the center of USS execution.  
This document outlines how USE connects to the surrounding layers and what 
guarantees it must uphold.

---

## 1. Incoming Interface (from USR)

USR emits a structured **ExecutionPlan**.  
USE must accept the plan with the following guarantees:

- **Deterministic structure**  
  Every step has a type: transform, evaluate, route, expand, compress.

- **Token-clean inputs**  
  All semantic units have been validated by UST invariants.

- **PlanBudget**  
  Maximum depth, time, and complexity allowed.

### Required Fields in Incoming Plans
- `plan_id`
- `steps[]`
- `constraints`
- `semantic_tokens[]`
- `posture` (global reasoning posture inherited from CE)
- `invariants_passed`

---

## 2. Internal Execution APIs

USE provides internal primitives — lightweight and deterministic — that CE and Domain Engines can call.

### Core Primitives
- `use.apply(token, op)`  
  Applies a semantic operation to a token.

- `use.route(step)`  
  Determines which execution micro-engine processes the step.

- `use.expand(token)`  
  Performs controlled semantic expansion (taxonomy, ontology, lineage).

- `use.reduce(token[])`  
  Condenses meaning into a smaller, stable semantic unit.

- `use.cast(token, type)`  
  Converts between token classes when allowed by invariants.

### Guarantees
- All operations must be **side-effect free**.  
- All state is passed explicitly through `exec_state`.  
- No mutation without an accompanying **Invariant Re-check**.

---

## 3. Outgoing Interface (to CE)

Once USE finishes execution:

It returns a **Reasoning Unit** (RU) to CE.

### RU Fields
- `context_window`
- `semantic_state`
- `confidence_model`
- `reasoning_trace`
- `invariant_status`

CE uses these fields to orient, evaluate, and justify the reasoning path.

---

## 4. Downstream Interface (to Domain Engines)

Domain Engines (SynCE, FinCE, QLE, etc.) plug in *after* CE.

USE must provide:
- Stable semantic structures for domain injection.
- A reasoning trace that the domain engine can extend.
- A contract for safe override of domain-specific inference.

### Domain Injection Points
- `inject.domain_knowledge(ru)`
- `inject.specialized_tokens(ru)`
- `inject.strategic_bias(ru)`  
  (e.g., financial constraints in FinCE, game-state rules in QLE)

Each injection must preserve:
1. **Token invariants**  
2. **Runtime posture**  
3. **Execution boundaries**  

---

## 5. Invariant Boundary Rules

USE is the first layer allowed to refuse execution if invariants fail.

It must halt when:
- Tokens violate semantic invariants.
- A plan step violates the execution graph.
- Posture is changed illegally mid-plan.
- Domain engine attempts an unauthorized override.

USE acts as the **semantic firewall** of USS.

---

## 6. Logging and Trace Output

USE outputs a structured, machine-readable trace:

- `trace_id`
- `step_history[]`
- `micro_engines_used`
- `branching_points`
- `posture_shifts`
- `invariant_checks`
- `final_state`

This trace becomes the backbone of explainability across the entire system.

---

## 7. Summary

USE is the translator, router, and execution governor of USS.

It stands between:
**USR’s deterministic plans**  
and  
**CE’s reasoning engine + Domain augmentation**

If UST is the “data model of meaning,”  
and USR is the “runtime circuit,”  
then USE is the “execution silicon.”

