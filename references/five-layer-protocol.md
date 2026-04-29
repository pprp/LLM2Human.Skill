# Five-Layer Output Protocol — LLM2Human Distillation

This document specifies the exact format, rules, and examples for the LLM2Human Learning Lens. It is the reference for Module 3 (Learning Lens Generator) in the distillation pipeline.

---

## Layer Overview

Every Learning Lens contains exactly these five layers. All five are mandatory unless the task is trivial (definition in the Calibration section below).

```
---
## Learning Lens

**Concepts:** [Layer 1]

**Method:** [Layer 2]

**Transfer:** [Layer 3]

**Boundary:** [Layer 4]

**Remember:** [Layer 5]
```

The `---` separator and `## Learning Lens` heading are required so the learning artifact is visually separated from the task answer.

---

## Layer 1 — Concepts

**Purpose:** Name the domain vocabulary and frameworks the solution relied on. This gives the user searchable terms and lets them recognize the pattern when they encounter it again.

**Format:** 2–4 bullet points. Each bullet names a concept and gives a one-line definition or context clue.

**Rules:**
- Name the concept explicitly — do not describe without naming
- Calibrate to user level: novice needs definition; expert needs only the name
- Include both the general concept (e.g., "amortized analysis") and the specific instance (e.g., "each element pushed/popped at most once")
- Do not list more than 4 concepts; if there are more, pick the 4 with highest transfer value

**Example (novice calibration):**
```
**Concepts:**
- **Monotonic stack** — a stack that maintains elements in sorted order by discarding
  elements that violate the ordering constraint as you push
- **Amortized O(n)** — though each push may cause multiple pops, each element is pushed
  and popped at most once total, making the total work O(n)
- **Invariant** — a condition that must remain true at every step of the algorithm;
  here: "the stack always contains candidates in increasing order"
```

**Example (expert calibration):**
```
**Concepts:**
- Monotonic stack, amortized O(n)
- Invariant maintenance as the correctness argument
```

---

## Layer 2 — Method

**Purpose:** Show the problem-solving *pattern* — not the solution itself, but the decomposition strategy that generated it. This is the worked-example layer: steps with their motivating principles.

**Format:** 3–5 numbered steps. Each step says what was done AND (for novice/practitioner) why.

**Rules:**
- Steps describe the *reasoning process*, not the implementation details
- Explain the decision at each step: why this choice, not just what was chosen
- Link each step to a named concept from Layer 1 where possible
- Expert calibration: compress to 2–3 steps, skip obvious motivation
- Do not repeat code or implementation specifics already in the answer

**Example:**
```
**Method:**
1. **Identify the goal structure** — asked "what is the nearest larger element?", which
   signals a comparison-as-you-go pattern rather than sorting or searching
2. **Choose monotonic stack** — because we need to track unresolved candidates
   (elements waiting for their answer) in a way that each new element can resolve many
   at once; the stack invariant does this in one pass
3. **Define the invariant** — elements on the stack are in increasing order; violating
   this means the new element is the answer for the popped element
4. **Verify amortization** — count pushes and pops: each element enters once, exits
   once, giving O(n) total regardless of inner loop iterations
```

---

## Layer 3 — Transfer

**Purpose:** Explicitly tell the user where else this pattern applies. This is the highest-value layer for long-term learning: it builds the schema, not just the solution.

**Format:** 2–3 concrete transfer contexts. Be specific — name the problem type, not just the concept.

**Rules:**
- Name analogous problem types, not just analogous concepts
- At least one transfer context should be outside the immediate domain (e.g., if the task was code, one transfer to system design or math)
- Phrase as "The same pattern applies when..." or "This works whenever..."
- Do not pad with obvious generalizations ("this applies to all sorting problems")

**Example:**
```
**Transfer:**
- **Next greater/smaller element variants** — temperature span, daily temperatures,
  stock span problem, largest rectangle in histogram all use the same stack-invariant structure
- **Dependency resolution with backtracking** — any time you process a stream and
  need to retroactively answer questions about previously seen elements
- **Compiler expression parsing** — operator precedence and parenthesis matching use
  the same "maintain a stack of unresolved contexts" pattern
```

---

## Layer 4 — Boundary

**Purpose:** State what assumptions the solution relies on, where it breaks down, and what requires human judgment. This layer protects against over-reliance and miscalibrated trust.

**Format:** 2–3 items. Format: assumption/limitation + consequence if violated.

**Rules:**
- Name *specific* assumptions, not generic disclaimers ("results may vary" is not a boundary)
- For each assumption, name what breaks if it is violated
- Include at least one "requires human judgment" item if applicable
- Do not artificially inflate uncertainty — only name real failure modes

**Example:**
```
**Boundary:**
- **Assumes single-pass is sufficient** — this approach works because the relationship
  "nearest larger element" is resolved as you go. Problems requiring look-ahead
  (e.g., "nearest larger within k steps") need a different structure
- **Integer/comparable elements only** — the monotonic invariant requires a total order.
  Partial orders (e.g., multi-key ranking with conflicts) need tie-breaking logic
- **Human judgment needed** — if the problem statement says "nearest" but the actual
  requirement is "within a time window", the stack approach needs adaptation that
  depends on the specific window semantics
```

---

## Layer 5 — Remember

**Purpose:** A micro-retrieval cue. Give the user exactly what they need to self-test recall 30 seconds, 30 minutes, or 3 days later. Based on the Retrieval Practice / Testing Effect.

**Format:** One of two forms:
1. **Three keywords** — "In 30 seconds you should still remember: [keyword 1], [keyword 2], [keyword 3]"
2. **One diagnostic question** — "Next time you see [trigger], ask yourself: [question]"

**Rules:**
- Keywords must be specific enough to trigger recall of the full concept (not "stack" but "monotonic stack invariant")
- The diagnostic question must be answerable only if the user understood the method — vague questions fail this test
- Do not combine both forms in the same Remember line; pick one
- For complex tasks, prefer the diagnostic question form

**Example (keyword form):**
```
**Remember:** monotonic stack, amortized O(n) by push/pop symmetry, invariant as correctness argument
```

**Example (diagnostic form):**
```
**Remember:** Next time you see "nearest/next greater/smaller element in a stream" — ask: what is my stack maintaining as an invariant, and when does a new element resolve old candidates?
```

---

## Calibration: When to Abbreviate

The full five-layer format is default. Abbreviate when:

| Condition | Adjustment |
|-----------|-----------|
| Task is trivial (< 3 reasoning steps) | Skip Method; keep Concepts + Remember |
| User is clearly an expert in the exact domain | Compress Concepts to names only; compress Method to 2 steps |
| User has explicitly said "just give me the answer" previously | Ask before adding Learning Lens, or omit with note |
| Same concept taught already this session | Skip that concept in the Concepts layer, note "as before:" |
| Deep teaching requested | Expand Method with full motivations; expand Transfer with 4+ contexts; add a Constructive Challenge after Remember |

**Trivial task definition:** A task where the solution is a single step, the concept is domain-common knowledge for this user's level, and no decision-making was involved.

---

## Constructive Challenge (Optional)

After the Remember line, add a Constructive Challenge only when:
- The user is clearly engaged and pushing their understanding
- The task involved a non-obvious design choice
- The user asked a follow-up "why" question during the task

**Format:**
```
**Challenge:** [One question the user should be able to answer if they understood the method — and that requires generating a response, not just recalling a fact]
```

**Example:**
```
**Challenge:** If the problem required "nearest larger element within k positions" instead of the full array, how would the algorithm change — and what would the new time complexity be?
```

This question is answerable only by someone who understood the invariant and can modify it. It pushes engagement from Active to Constructive (ICAP framework).

---

## Full Example — Complete Learning Lens

**Task:** User asked for help writing a SQL query that finds the second-highest salary in an employee table.

```
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

---
## Learning Lens

**Concepts:**
- **Subquery (scalar)** — a SELECT inside another SELECT that returns a single value; here it computes the maximum salary for the WHERE clause to filter against
- **Exclusion-then-maximize pattern** — exclude the known maximum, then find the new maximum; a generalizable technique for "nth largest/smallest" problems
- **Correlated vs. non-correlated subquery** — this subquery is non-correlated (runs once, not once per row), so it is efficient at O(n) rather than O(n²)

**Method:**
1. **Identify the goal** — "second highest" means "highest of everything that isn't the highest", which maps to exclude-then-maximize
2. **Compute the maximum first** — inner subquery gets the ceiling to exclude
3. **Exclude and re-maximize** — outer query finds the highest value below that ceiling
4. **Verify edge case handling** — if all salaries are equal or the table has one row, this returns NULL; decide if that's acceptable before shipping

**Transfer:**
- **Nth largest in any ordered set** — wrap this pattern in one more layer to get the 3rd, 4th, etc. highest (or use `ORDER BY ... LIMIT 1 OFFSET n-1` for clarity)
- **Excluding known extremes in analytics** — "best-performing product excluding the flagship" uses the same structure
- **Median and percentile queries** — similar exclusion logic, though typically solved with `PERCENTILE_CONT` in SQL dialects that support it

**Boundary:**
- **NULL handling not explicit** — if salary can be NULL, `MAX()` ignores NULLs silently; verify this is the intended behavior
- **Tied second-highest** — this returns one value even if multiple employees share the second-highest salary; `GROUP BY` or `DENSE_RANK()` would be needed if you want all tied employees
- **Performance on large tables** — the double full-scan is fine for < 1M rows; at scale, a window function (`DENSE_RANK() OVER (ORDER BY salary DESC)`) is faster with an index

**Remember:** exclusion-then-maximize pattern, non-correlated scalar subquery, NULL and tie edge cases
```

---

## Anti-Patterns to Avoid

| Anti-pattern | Problem |
|---|---|
| Restating the answer in different words | Adds cognitive load with no new information |
| "This works because the code is correct" | Not a pedagogical artifact — circular |
| Generic transfer: "applies to all sorting" | Too vague to build a schema |
| Boundary: "results may vary" | Not a specific assumption or failure mode |
| Remember: "understand the algorithm" | Not a retrieval target — untestable |
| Concepts layer with 8 bullet points | Violates Cognitive Load Theory; compress to 4 max |
| Method that just reprints the code steps | Method should describe reasoning, not re-narrate implementation |
