# User Profiles — LLM2Human Distillation Calibration

This document specifies how the Learning Lens adapts to the user's apparent expertise. Calibration is the most important quality lever: a lens that is too basic is condescending; one that is too advanced is incomprehensible. Both fail the learning goal.

---

## Calibration Signals

The agent infers expertise from conversational evidence, not from explicit declaration. Signals to read:

### Strong signals (update calibration immediately)

| Signal | Update |
|--------|--------|
| User uses domain terminology correctly | Raise calibration for that domain |
| User uses domain terminology incorrectly | Lower calibration for that domain |
| User asks "what is X?" where X is a basic concept | Lower calibration; define basics |
| User catches a subtle error or suggests an optimization | Raise calibration significantly |
| User says "I'm a [role]" or "I've been doing this for N years" | Use role/years as prior; still watch for gaps |
| User asks for explanation of a concept without knowing the name | Mixed signal: knows they don't know, calibrate to practitioner |
| User's code/writing shows professional patterns | Raise calibration |
| User's code/writing shows beginner patterns (no error handling, poor naming) | Lower calibration |

### Weak signals (adjust gradually)

| Signal | Update |
|--------|--------|
| Short, confident questions | Slight raise |
| Long, hedged questions with many "maybe?" qualifiers | Slight lower |
| Copy-pasted error messages with no context | Neutral; gather more evidence |
| Question that could only come from prior knowledge | Raise for that specific area |

### Domain-specific calibration

Calibration is per-domain, not per-user globally. A user can be an expert in Python and a novice in Kubernetes. Calibrate independently:
- Language/syntax level
- Framework level  
- Architecture/system design level
- Domain theory level (algorithms, ML theory, etc.)

---

## Three Calibration Levels

### Level 1 — Novice

**Characteristics:** Limited or no prior exposure to the domain concepts used in this task. Does not know the vocabulary. Has not formed schemas for the problem type.

**Learning Lens adjustments:**

**Concepts layer:**
- Define every named concept in one sentence
- Use analogies to familiar domains ("a stack is like a stack of plates — you can only add or remove from the top")
- Include one concrete small example for each concept
- Maximum 3 concepts (novices need depth, not breadth)

**Method layer:**
- Use 4–5 steps
- Explain the motivation for each step ("we do this because...")
- Avoid abbreviations and undefined acronyms
- State the invariant explicitly and name it as such

**Transfer layer:**
- Focus on the most obvious, closest transfer contexts
- Name a familiar analogy from everyday experience if possible
- Avoid transferring to domains the user likely doesn't know

**Boundary layer:**
- Name one assumption in terms the user will immediately recognize
- Avoid assumptions that require advanced knowledge to understand
- One "human judgment needed" item to build appropriate caution

**Remember line:**
- Three keywords that are comprehensible without looking them up
- Or a question that tests the core insight, not a subtle edge case

**Tone:** Patient, encouraging, explicit. Never assume they remember the definition from two turns ago.

---

### Level 2 — Practitioner

**Characteristics:** Knows the domain vocabulary. Has encountered the problem class before. Can read the solution code or architecture diagram without explanation. Wants to understand the *why* and the *trade-offs*, not the *what*.

**Learning Lens adjustments:**

**Concepts layer:**
- Name concepts without defining the obvious ones
- Focus on concepts they know the name of but may not know the precise formal definition ("you probably know this as X — the formal name is Y")
- 2–4 concepts; prioritize concepts that explain the design choice over alternatives

**Method layer:**
- 3–4 steps
- Emphasize decision criteria: "we chose X over Y because..."
- Include the trade-off that was accepted
- Skip motivation for steps that are obvious at this level

**Transfer layer:**
- Include one non-obvious transfer — a context outside the immediate domain or problem class
- Name the specific situations where the pattern holds AND where it breaks (bridges to Boundary)

**Boundary layer:**
- All three items at full specificity
- Include one subtle assumption they might miss even as a practitioner

**Remember line:**
- Three keywords or a diagnostic question
- The question should test trade-off understanding, not just recall

**Tone:** Collegial. Assume they are capable of following technical reasoning. Challenge their thinking slightly.

---

### Level 3 — Expert

**Characteristics:** Has deep knowledge of the domain. Likely knows multiple approaches to this problem class. Will notice if the agent makes a suboptimal choice. Interested in edge cases, formal properties, and the space of alternatives.

**Learning Lens adjustments:**

**Concepts layer:**
- Names only, no definitions
- Optionally include the formal name if the common name was used ("this is the amortized analysis by the potential method")
- 2 concepts maximum; if the expert knows them all, skip this layer and note "all concepts in use here are within your expertise"

**Method layer:**
- 2–3 steps, compressed
- Focus on the non-obvious step: the one a practitioner might miss
- Name the alternative approaches considered and why they were not chosen
- If the solution made a suboptimal choice for simplicity, name it explicitly

**Transfer layer:**
- Push to adjacent fields or theoretical generalizations
- "This is an instance of [general principle] which also appears in [non-obvious field]"
- Name a known harder variant the expert might want to explore

**Boundary layer:**
- Prioritize subtle or counterintuitive failures — obvious ones are beneath this level
- Include a formal assumption (e.g., "assumes total order exists" rather than "assumes elements are comparable")
- If there's an open question or known research problem in this area, name it

**Remember line:**
- A diagnostic question that would be considered a hard interview question
- Or the name of a paper/concept the expert can look up for depth

**Tone:** Peer-to-peer. Be direct about trade-offs. Don't hedge unnecessarily. The expert will push back if they disagree — that is the goal.

---

## When to Ask Instead of Guess

Explicit calibration check is appropriate when:
- The task is long and the learning section would differ significantly between levels
- The user's signals are contradictory (e.g., expert vocabulary but novice code structure)
- The user has requested deep teaching, suggesting they want a specific depth

**Calibration prompt:**
```
Before I add the Learning Lens — are you already familiar with [key concept], or would you like me to include a brief explanation?
```

Keep it to one question. Multiple calibration questions interrupt the flow.

---

## Special Cases

### User explicitly sets their level
If the user says "explain as if I'm a beginner" or "assume I know [X]", override all inferred signals and use that level for the rest of the session.

### User's level shifts mid-session
Re-calibrate forward (raise or lower) when a strong signal appears. Do not wait until the next task.

### User asks for the lens to be skipped
Respect this without pushback. Note "Learning Lens skipped per your preference." If they later ask for it, resume without reminder that they previously skipped it.

### User is a teacher/educator preparing material
If the user's context indicates they are preparing learning material for others (not learning themselves), switch mode: the Learning Lens should be written for the user's *students*, at their level — not for the user themselves. Ask for the audience level if not clear.

---

## Quick Reference Table

| Dimension | Novice | Practitioner | Expert |
|-----------|--------|-------------|--------|
| Concepts layer | Name + define + example | Name + brief context | Name only (or skip) |
| Method layer | 4–5 steps + motivation | 3–4 steps + trade-offs | 2–3 steps + non-obvious only |
| Transfer layer | Closest + familiar analogy | Including non-obvious | Adjacent fields + theory |
| Boundary layer | One recognizable assumption | Full specificity | Subtle + formal + open questions |
| Remember line | Comprehensible keywords | Keywords or diagnostic | Hard diagnostic or paper name |
| Max concepts | 3 | 4 | 2 (or skip) |
| Tone | Patient, explicit | Collegial, trade-off focused | Peer, direct, challenging |
