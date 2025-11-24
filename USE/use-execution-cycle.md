# USE Execution Cycle

The Universal Semantic Engine (USE) is the **semantic executor** in the USS stack.

Where:

- **UST** defines the stable units of meaning.  
- **USR** plans over those units (ExecutionPlans).  
- **USE** *runs* those plans deterministically.  
- **CE / Domain Engines** provide the cognition and domain-specific reasoning that USE calls into.

This document describes the **execution cycle** USE follows for every request.

---

## 1. Inputs, Outputs, and Invariants

### Inputs

USE receives:

- A **UST payload**  
  Canonical semantic tokens (and structures) produced upstream.

- A **USR ExecutionPlan**  
  A structured plan with:
  - `plan_id`
  - ordered `steps`
  - optional `constraints` (time, cost, depth)
  - required **invariants** (things that must remain true)

- A **runtime context handle**  
  References to:
  - conversation / task history
  - operator / posture settings
  - safety / governance flags

### Outputs

USE produces:

- A **semantic result**  
  Transformed USTs (e.g., answers, summaries, decisions, proofs).

- A **trace**  
  Step-by-step log of the execution, with:
  - which modules ran
  - which CE / Domain Engines were invoked
  - intermediate semantic states (or hashes of them)

- A **status record**  
  - `SUCCESS`, `SOFT_FAIL`, or `HARD_FAIL`  
  - any invariant violations or safety interventions

### Invariants

During execution, USE enforces:

- **Type Safety** — All operations respect UST type contracts.  
- **Plan Integrity** — Steps must follow the plan unless an approved deviation rule triggers.  
- **Determinism** — Given the same inputs + plan + configuration, USE produces the same result.  
- **Governance Hooks** — Safety, ethics, and operator rules are enforced at each boundary.

---

## 2. High-Level Execution Loop

At a high level, USE runs this cycle:

1. **Intake & Validation**  
2. **Context Binding**  
3. **Plan Compilation**  
4. **Step-by-Step Execution**  
5. **Error Handling & Recovery**  
6. **Finalization & Logging**

Each stage is deterministic inside USE, even if the CE or Domain Engines may use probabilistic methods internally.

---

## 3. Stage 1 — Intake & Validation

**Goal:** Ensure USE only executes *well-formed*, *governed* plans.

Steps:

1. **Schema Check**  
   - Validate ExecutionPlan structure (required fields present, step formats correct).  

2. **Invariants Load**  
   - Load invariants from:
     - USR plan metadata  
     - stack-level governance (e.g., safety rules, CAP, Binder rules)  

3. **Feasibility Check**  
   - Quickly reject obviously impossible plans:
     - unknown operations
     - incompatible UST types
     - forbidden modules (e.g., disabled domain engine)

4. **Admission Decision**  
   - If validation fails → return `HARD_FAIL` with reasons.  
   - If validation passes → move to context binding.

---

## 4. Stage 2 — Context Binding

**Goal:** Attach the right semantic and situational context to the plan.

Steps:

1. **UST Resolution**  
   - Map UST IDs in the plan to concrete semantic structures in memory.

2. **Context Attach**  
   - Attach:
     - prior turns / documents relevant to the task
     - operator preferences (e.g., “explain in plain English”)
     - active cognitive posture (Analytic, Exploratory, etc., if configured)

3. **Safety Scope**  
   - Determine the **allowed scope** of reasoning:
     - sensitive domains
     - red-line topics
     - maximum depth/complexity allowed for this request

Result: USE now has a **bound ExecutionPlan** — same structure, but anchored in a concrete execution context.

---

## 5. Stage 3 — Plan Compilation

**Goal:** Turn abstract plan steps into executable micro-operations.

For each step in the plan:

1. **Operation Lookup**  
   - Map step name (e.g., `EXTRACT_FEATURES`, `COMPARE`, `SUMMARIZE_RISKS`)  
     to an internal **execution primitive**.

2. **Module Routing**  
   - Decide which module should run it:
     - internal USE primitive
     - CE call (cognitive reasoning)
     - Domain Engine call (e.g., SynCE, FinCE, QLE)
     - external tool / API (if allowed)

3. **Precondition Attachment**  
   - Define:
     - required input UST types
     - invariants that must hold before execution

4. **Postcondition Specification**  
   - Define:
     - expected output UST types
     - new invariants (e.g., “risk scores must be numeric between 0 and 1”)

The result is a **CompiledPlan** that is mechanically executable.

---

## 6. Stage 4 — Step-by-Step Execution

**Goal:** Run the compiled plan deterministically, updating semantic state.

For each step `s` in `CompiledPlan.steps`:

1. **Precondition Check**  
   - Verify:
     - required inputs exist
     - UST types match
     - no invariant is already broken

   If a precondition fails:
   - if recoverable → branch to recovery rule (e.g., re-query CE with stricter constraints)  
   - else → mark `SOFT_FAIL` or `HARD_FAIL` and exit.

2. **Module Invocation**

   Depending on `s.module_type`:

   - **USE Primitive**  
     - Internal transformations (e.g., filtering, merging USTs, sorting, simple numeric ops).

   - **CE Call**  
     - Hand off USTs + instructions to the Cognition Engine.  
     - CE performs reasoning using the configured cognitive posture.  
     - USE receives back:
       - transformed USTs
       - reasoning traces (if enabled)

   - **Domain Engine Call**  
     - Invoke SynCE / FinCE / QLE or other engines on structured USTs.  
     - Domain Engine applies domain logic (e.g., finance rules, legal constraints).  
     - USE receives domain-specific results as UST structures.

3. **Postcondition & Invariant Check**

   After module completion:

   - Validate:
     - output USTs match expected types
     - new invariants hold (e.g., monotonicity, bounds, safety rules)
   - If a violation occurs:
     - attempt local correction (if allowed)
     - otherwise escalate to recovery or fail the plan

4. **State Update**

   - Persist new semantic state into the **execution context**:
     - intermediate results keyed by step
     - any CE/Domain Engine annotations
   - Record the step in the execution trace.

The loop continues until:

- all steps have been run successfully, or  
- a fail state terminates execution.

---

## 7. Stage 5 — Error Handling & Recovery

USE distinguishes between **soft** and **hard** failure modes.

### Soft Failures

Issues where partial progress is still usable:

- missing optional data  
- non-critical invariant warnings  
- CE / Domain Engine returns “low confidence” tag

Handling:

- mark step as `SOFT_FAIL` with details  
- depending on configuration:
  - continue with downgraded reasoning, or
  - attempt one recovery pass (e.g., alternate path or simplified plan)

### Hard Failures

Critical violations:

- broken type invariants  
- governance / safety violation  
- repeated CE / Domain Engine errors  
- unstoppable plan divergence

Handling:

- abort remaining steps  
- mark overall plan as `HARD_FAIL`  
- return:
  - best-effort partial results (if safe), plus
  - explicit failure report for inspection

---

## 8. Stage 6 — Finalization & Logging

When execution ends, USE:

1. **Collects Final Result**

   - Selects the designated output UST(s) from the semantic state.  
   - Optionally normalizes them for downstream formatting.

2. **Assembles Execution Trace**

   - Ordered list of steps with:
     - module calls
     - inputs and outputs (or hashes)
     - any warnings, soft failures, or recovery paths

3. **Applies Governance Hooks**

   - Run final checks:
     - CAP / Binder / ethical constraints (if integrated)
     - redaction or masking rules for sensitive data

4. **Emits Response**

   - Returns to the caller (USR / higher orchestration layer):
     - result USTs
     - status (`SUCCESS`, `SOFT_FAIL`, `HARD_FAIL`)
     - summarized trace / rationale

5. **Persists Optional Telemetry**

   - Log run metadata for:
     - performance profiling
     - semantic debugging
     - future tool / Codex integration

---

## 9. Relationship to the USS Stack

Putting it all together:

1. **UST**  
   - Defines *what* the semantic atoms are.

2. **USR**  
   - Decides *what to do* with those atoms (plans and invariants).

3. **USE**  
   - Decides *how to run* that plan, step by step, reliably.

4. **CE / Domain Engines**  
   - Provide *why this answer* — the cognitive or domain reasoning behind each step.

USE is the bridge between **abstract semantic planning** (USR) and **concrete semantic reasoning** (CE + Domain Engines).  
The execution cycle above is how that bridge stays deterministic, inspectable, and safe.
