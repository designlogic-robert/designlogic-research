# UST Token Schema (Working Draft)

This note defines the **canonical fields** and **structural rules** for all Universal Semantic Tokens (USTs) used in the USS stack.

The goal is to make USTs:

- **Typed** – every token has a clear semantic category.
- **Composable** – tokens can be combined into larger structures without ambiguity.
- **Portable** – any engine that understands the schema can consume USTs, regardless of surface language.

---

## 1. Core Principles

1. **Surface-agnostic**  
   USTs represent *meaning*, not text. No field depends on a particular human language.

2. **Type-first design**  
   Every UST declares its type explicitly from a shared type system (concept, event, relation, modality, etc.).

3. **Invariant-backed**  
   Each token type has invariants (things that must always be true) that USR/USE can validate at runtime.

4. **Auditability**  
   Tokens carry enough metadata for provenance, versioning, and safety checks.

---

## 2. Minimal UST Envelope

Every UST instance MUST include at least:

- `token_id` – Globally unique identifier (UUID or content hash).
- `family` – One of: `semantic`, `teleo`, `trade` (extensible).
- `type` – Token type within the family, e.g. `CONCEPT`, `EVENT`, `RELATION`, `GOAL`, `OBLIGATION`, `POSITION`.
- `payload` – Family-specific structured content (see below).
- `invariants` – List of invariant ids this token asserts (e.g. `INV.NON_NEGATIVE_AMOUNT`).
- `scope` – Where this token is valid (domain, jurisdiction, time span, etc.).
- `version` – Schema version for this token’s family.
- `provenance` – Source info (engine, parser, human annotator, etc.).
- `confidence` – Optional numeric or categorical confidence score.

---

## 3. Semantic Token Payload Shape

For **semantic tokens** (family = `semantic`), the payload is usually:

- `concept` – Canonical concept id (e.g. `MEDICAL_PRACTITIONER`).
- `roles` – Role-filler map for predicate–argument structure.  
  Example:

  ```json
  {
    "predicate": "DIAGNOSE",
    "roles": {
      "agent": "PHYSICIAN#123",
      "patient": "PERSON#456",
      "condition": "DISEASE#COVID19"
    }
  }
temporal – Optional temporal qualifiers (before/after, duration, interval).

modality – Optional modality (belief, obligation, permission, risk).

Key invariant: role completeness – a token must either provide all required roles for its predicate or mark missing roles as intentionally absent/unknown.

---

## 4. Teleo Token Payload Shape
For teleo tokens (family = teleo), payload focuses on goals and trajectories:

goal – Canonical goal id or description.

agent – Who this goal belongs to (system/user/organization).

constraints – Hard boundaries (safety, ethics, regulation).

metrics – How success is measured.

horizon – Time/step horizon for the goal.

Teleo tokens are what USE/CE use to evaluate plans against desired outcomes.

---

## 5. Trade Token Payload Shape
For trade tokens (family = trade), payload captures positions and flows:

instrument – What’s being traded (asset id, contract id, etc.).

side – LONG / SHORT / FLAT.

size – Notional amount.

price – Price or pricing rule.

constraints – Risk / compliance constraints.

context – Market / venue / regime info.

Trade tokens are specialized for FinCE and any economic/market reasoning engines.

---

## 6. Cross-Family Shared Fields
All families share:

links – References to other USTs (graph edges).

tags – Free-form but controlled vocabulary labels.

safety_flags – Markers added by safety systems (e.g. RISK_HIGH, AMBIGUOUS_CONTEXT).

These support graph queries, safety passes, and orchestration across engines.

---

## 7. Schema Extension Rules
To add new token types or families:

Define invariants for the new type.

Specify payload structure (fields, allowed values).

Register the new type/family in UST type registry.

Provide migration rules from older versions if needed.

No engine may emit a UST that violates the core envelope fields or its declared invariants.

---

## 8. Relationship to the Formal Papers
The Semantic Tokens v1.0 paper defines the theory and canonical examples.

Teleo Tokens v1.0 and Trade Tokens v1.0 specialize the schema for goals and markets.

This note is the practical schema reference that USR/USE/CE will assume when interpreting UST instances.

Future changes to the schema MUST be versioned and accompanied by migration notes.