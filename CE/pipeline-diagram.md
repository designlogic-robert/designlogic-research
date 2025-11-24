```
+---------------------------+
| UST – Universal Tokens    |
|  (semantic meaning units) |
+-------------+-------------+
              |
              v
+---------------------------+
| USR – Semantic Runtime    |
|  deterministic meaning    |
|  flow + invariants        |
+-------------+-------------+
              |
              v
+--------------------------------------+
| USE – Semantic Executor              |
|  interprets tokens, sequences ops    |
+-------------+------------------------+
              |
              v
+----------------------------------------------+
| CE – Cognition Engine                        |
|----------------------------------------------|
| 1. Orient   → read context, grounding        |
| 2. Evaluate → relevance, risk, confidence    |
| 3. Model    → build internal reasoning shape |
| 4. Select   → choose strategy / path         |
| 5. Justify  → “why this answer?”             |
| 6. Finalize → produce reasoning unit         |
+-------------+--------------------------------+
              |
              v
+---------------------------+
| Domain Engine (SynCE,     |
|  FinCE, QLE, etc.)        |
|  adds domain knowledge    |
+-------------+-------------+
              |
              v
+---------------------------+
| Output (to user/system)   |
+---------------------------+
```
The Cognition Engine sits between USE and the Domain Engine. USE sends structured semantic operations upward, and CE applies a reasoning process to them. It thinks through a problem in six stages—orient, evaluate, model, select, justify, finalize—and hands the resulting cognitive directive to the Domain Engine. This ensures every domain (SynCE, FinCE, QLE) thinks in a stable and predictable way.