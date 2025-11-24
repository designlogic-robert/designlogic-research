# UST Lifecycle & Versioning

This note explains how USTs are **created, validated, updated, and retired** across the USS stack, and how versioning keeps engines interoperable.

---

## 1. Stages of a UST

1. **Birth (Parsing / Construction)**  
   - Input: text, event streams, structured data, or engine-internal reasoning.  
   - Output: candidate USTs with confidence scores.  
   - Responsibility: parsers, mapping engines, or CE itself.

2. **Validation (USR / USE)**  
   - Type checks: does the token match its declared type & family?  
   - Invariant checks: are all invariants satisfied?  
   - Scope checks: is the token valid in this domain/jurisdiction/horizon?

3. **Use in Planning / Reasoning**  
   - Tokens are consumed by CE, domain engines (SynCE, FinCE, QLE, etc.), or tools.  
   - Transformations must preserve type correctness and invariants, or explicitly mark changes.

4. **Update / Refinement**  
   - New evidence may refine fields (e.g. diagnosis updated, trade size adjusted).  
   - Updates must increment a **revision id** and keep provenance history.

5. **Archival / Retirement**  
   - Tokens can be archived when out of scope or expired.  
   - Safety systems may mark tokens as obsolete or unsafe (`status = "DEPRECATED"`).

---

## 2. Versioning Dimensions

USTs have three relevant versions:

1. **Family schema version**  
   - `semantic` v1.0, `teleo` v1.0, `trade` v1.0, etc.  
   - Changes here affect structure and required fields.

2. **Token instance revision**  
   - `rev`: incremented whenever the *same* real-world entity/statement is updated.  
   - Engines can choose to keep or discard prior revisions.

3. **Invariant set version**  
   - Invariant definitions may evolve (e.g., new regulatory rules).  
   - USTs reference invariant ids that are resolved against an invariant registry.

---

## 3. Compatibility Rules

- **Forward compatibility**  
  Engines built for schema v1.0 should be able to *ignore* unknown optional fields introduced in v1.1+.

- **Backward compatibility**  
  When deprecating a field, provide:
  - Migration mapping (old → new)
  - Fallback behavior for engines that still expect the old field

- **Breaking changes**  
  Only allowed on major version bumps (`v1.x` → `v2.0`) and must be accompanied by:
  - Migration spec  
  - Updated invariant registry  
  - Test suite updates

---

## 4. Token Identity & Hashing

To keep reasoning reproducible:

- `token_id` SHOULD be a **content-addressed id** (hash of canonical payload + type + family).
- Revisions may either:
  - Keep the same `token_id` with a separate `rev` field, or
  - Use new `token_id` and link via `previous_id`.

This makes it possible to:

- Re-run reasoning chains on the exact same tokens.
- Detect accidental changes to critical semantics.

---

## 5. Lifecycle Hooks for Engines

Engines integrating USTs should expose hooks:

- `on_token_birth(ust)` – initial validation, safety tagging.
- `on_token_update(old_ust, new_ust)` – diffing, invariant re-checks.
- `on_token_archive(ust)` – cleanup, reference updates.
- `on_version_mismatch(ust, expected_schema)` – fallback or error.

These hooks allow USR/USE to coordinate **semantic hygiene** across the ecosystem.

---

## 6. Relationship to LLpL / Persistence

In a full USS deployment:

- USTs are stored in a persistence layer (LLpL-like) with:
  - Content hashes
  - Version metadata
  - Provenance trajectories

Lifecycle & versioning rules in this note act as the **contract** between USTs and persistence.

---

## 7. Open Questions / R&D Notes

- How aggressively should old UST revisions be retained vs. garbage-collected?
- What’s the minimal metadata required for regulatory-grade audit trails?
- How do we represent *soft* invalidation (e.g., “no longer trusted” but still historically relevant)?

These will be answered as more domain engines (SynCE, FinCE, QLE) are prototyped against UST v1.x.