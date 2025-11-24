# CE Deterministic Cycle  
*Working Draft — Universal Semantic System (USS)*

The Cognition Engine (CE) runs a **deterministic micro-cycle** over semantic inputs.  
Where LLMs operate as opaque probability machines, CE exposes a stable, inspectable reasoning loop.

At the system level, the pipeline is:

> **UST → USR → USE → CE → Domain Engine → Output**

Within **CE**, the internal loop is:

> **1. Orient → 2. Evaluate → 3. Model → 4. Select → 5. Justify → 6. Finalize**

Each step consumes and emits **typed semantic state** rather than raw text.

---

## 0. Inputs and Pre-Conditions

CE receives from USE:

- a **semantic task description** (USTs + metadata)  
- the **active Cognitive Posture** (from `ce-cognitive-postures.md`)  
- **constraints and invariants** (risk level, domain, caps, time/complexity budget)  
- optional **prior reasoning trace** (for multi-step plans)

CE assumes:

- all semantic units are valid USTs  
- USR has already routed the task to the correct engine chain  
- USE will respect CE’s decisions about path selection and justification level

---

## 1. Orient — Establish Context and Grounding

**Goal:** Figure out *what kind of thinking* is required before actually thinking.

Operations:

- Identify task type (explain, decide, compare, plan, generate options, etc.)  
- Inspect UST set for:
  - domain markers  
  - critical entities and relations  
  - time, risk, and obligation semantics  
- Activate or refine the **Cognitive Posture**:
  - Analytical, Integrative, Pragmatic, Exploratory, Reflective, Domain-Adaptive  

Outputs:

- `ContextFrame` (what the problem is “about”)  
- `FocusSet` (which USTs matter most)  
- `PostureState` (chosen posture + parameters)

Plain English: this is CE asking “What am I actually being asked to do, and how serious is it?”

---

## 2. Evaluate — Relevance, Risk, and Weight

**Goal:** Decide what matters, how much, and why.

Operations:

- Score USTs for:
  - **Relevance** (to the task and context)  
  - **Risk impact** (legal, financial, safety, etc.)  
  - **Uncertainty** (ambiguity, missing context, conflicting signals)  
- Apply invariants:
  - “never ignore high-risk USTs”  
  - “flag conflicts between critical USTs”  
- Optionally request more information (via USE) if gaps are too large.

Outputs:

- `WeightedUSTGraph` (USTs + weights + uncertainty)  
- `RiskProfile` (low/med/high, with reasons)  
- `AttentionMask` (what to emphasize, what to down-weight)

Plain English: this is CE saying “Given everything here, *what should I pay attention to* and *how careful should I be*?”

---

## 3. Model — Construct Semantic Structure

**Goal:** Build an internal reasoning model that can actually be operated on.

Operations:

- Compose USTs into structured forms:
  - causal chains  
  - decision trees  
  - constraint networks  
  - scenario graphs  
- Enforce type constraints:
  - only valid semantic compositions allowed  
  - catch “nonsense joins” early (e.g., mixing incompatible types)  
- Choose **model form** based on posture + domain:
  - Analytical → proof/derivation style  
  - Integrative → multi-domain mapping  
  - Pragmatic → compressed decision tree  
  - Exploratory → branching scenario graph  

Outputs:

- `ReasoningModel` (formal structure over USTs)  
- `ModelAssumptions` (explicit approximations / shortcuts taken)

Plain English: this is CE building a map of “how things relate and what might happen if we act.”

---

## 4. Select — Choose Path / Strategy

**Goal:** Decide *which route through the model* to commit to.

Operations:

- Enumerate candidate reasoning paths:
  - different sequences of UST transformations  
  - different policy choices or options  
- Score paths using:
  - `RiskProfile`  
  - domain preferences  
  - posture rules (e.g., Analytical prefers conservative, safer paths)  
- Apply constraints:
  - time/complexity budgets  
  - required justification depth  
- Pick the **Primary Path** and optionally a **Fallback Path**.

Outputs:

- `SelectedPath` (ordered steps to execute)  
- `PathRationale` (why this path, not others)  

Plain English: this is CE saying “Of all the ways to think about this, *we’re going with this specific route* for these reasons.”

---

## 5. Justify — Make Reasoning Inspectable

**Goal:** Attach human-grade explanation to the chosen path.

Operations:

- Generate **step-level rationales**:
  - for each node/edge in `SelectedPath`, describe:
    - what is being done  
    - which USTs are involved  
    - which invariants or heuristics applied  
- Highlight key trade-offs:
  - “we favored safety over speed here”  
  - “we accepted this approximation to stay within budget”  
- Produce a **Posture-aware explanation style**:
  - Analytical → more formal, precise  
  - Pragmatic → shorter, action-oriented  
  - Reflective → explicitly self-critical  

Outputs:

- `JustificationTrace` (explainable reasoning log)  
- `DecisionSummary` (short, posture-tuned overview)

Plain English: this is CE leaving a “paper trail” so humans and tools can see how it got there.

---

## 6. Finalize — Package Reasoning Unit for USE + Domain Engine

**Goal:** Turn internal reasoning into a clean artifact that downstream engines can trust.

Operations:

- Normalize final state into a **ReasoningUnit**:
  - final conclusion or recommendation  
  - structured evidence references  
  - validity scope and caveats  
- Tag with:
  - posture used  
  - risk level and residual uncertainty  
  - any violated or softened invariants  
- Emit control hints for USE:
  - whether to call a specific domain engine  
  - whether to request human review  
  - whether to store to long-term semantic memory  

Outputs:

- `ReasoningUnit` (final semantic product)  
- `ControlHints` (how the rest of the stack should treat this result)

Plain English: this is CE saying “Here is my answer, here’s how I got it, here’s how confident I am, and here’s how you should use it.”

---

## Determinism: What It Means Here

CE’s cycle is **deterministic over semantic state**, not over raw text.

- Given:
  - the same UST set  
  - the same posture  
  - the same invariants and budgets  
- CE will produce:
  - the same `ReasoningModel`  
  - the same `SelectedPath`  
  - the same `ReasoningUnit` + `JustificationTrace`

The surface-form output may still vary when rendered as language by downstream models, but the **semantic decision** is stable and auditable.

---

## Relationship to Other USS Components

- **UST** defines *what* CE can reason *about*.  
- **USR** determines *where* the reasoning should happen.  
- **USE** manages *how* CE is orchestrated and sequenced.  
- **CE** decides *how to think* in a structured, posture-aware, deterministic way.  
- **Domain Engines** (SynCE, FinCE, QLE, etc.) supply specialized world-knowledge and behaviors on top of CE’s reasoning outputs.

This document should be read together with:

- `ce-cognition-overview.md`  
- `ce-cognitive-postures.md`  
- `pipeline-diagram.md`
