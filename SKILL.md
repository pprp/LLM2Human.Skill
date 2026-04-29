---
name: llm2human
description: >
  LLM2Human Distillation layer — transforms agent task solutions into human-learnable artifacts.
  Wraps any completed task with a "Learning Lens": concepts used, problem-solving pattern,
  transferable heuristic, failure modes, and a reflection cue to close the learning loop.
  Trigger on: "teach me", "explain your thinking", "help me understand how you did that",
  "I want to learn from this", "explain the approach", "walk me through your reasoning",
  "what concepts are at play", "learning mode", "distill this", "llm2human", "learning lens",
  "explain like I should learn it", "how would I do this myself next time",
  "knowledge distillation", "what did you learn from solving this",
  "我想学习", "教我", "解释思路", "学习模式".
  Do NOT trigger for pure task-execution requests where the user only wants results and
  has shown no interest in the methodology behind the answer.
---

# LLM2Human Distillation

> **Core goal:** Not to show more reasoning, but to convert agent work into learnable, transferable, and cognitively efficient artifacts.

This skill implements an output transformation layer based on six learning-science principles:
Cognitive Apprenticeship, Cognitive Load Theory, Worked Examples, ICAP Framework,
Retrieval Practice, and XAI Mental Models.

Full theory reference: `references/pedagogical-theories.md`
Output format spec: `references/five-layer-protocol.md`
Expertise adaptation: `references/user-profiles.md`

---

## When to Activate

Activate when the user signals a learning intent:
- Explicit phrases: "teach me", "explain your approach", "learning mode", "llm2human", "how would I do this myself"
- Implicit signals: asking follow-up questions about *why* rather than *what*, expressing unfamiliarity with the domain, asking for the reasoning process
- Task types that naturally carry high transfer value: algorithm design, architecture decisions, debugging strategies, research methodology, framework selection

Do **not** activate when:
- The user only wants the result and has given no signal they want to learn
- The task is trivial or the methodology is already obvious to the user's level
- The user has explicitly asked to skip the Learning Lens this session

---

## Execution Model

### Step 1 — Complete the Task

Solve the user's request fully and correctly. The Learning Lens is appended *after* the complete answer, never instead of it. Task utility comes first.

### Step 2 — Assess the User's Expertise

Read contextual signals to calibrate depth:
- **Novice:** Use analogies, define jargon, name each step, give concrete examples
- **Practitioner:** Skip basics, emphasize decision criteria and trade-offs
- **Expert:** Focus on the non-obvious: edge cases, assumptions, failure modes, where the pattern breaks down

See `references/user-profiles.md` for detailed calibration rules.

### Step 3 — Run the Distillation Pipeline

Internally execute these four modules before writing the Learning Lens:

**Module 1 — Knowledge Extractor**
Identify what learnable knowledge the task solution contains:
- Domain concepts invoked (named vocabulary, frameworks, models)
- Procedural steps (the how-to sequence)
- Decision criteria (why this approach over alternatives)
- Heuristics used (rules of thumb that are widely reusable)
- Failure modes (what goes wrong if you apply this incorrectly)
- Transfer patterns (where else this exact reasoning applies)

**Module 2 — Pedagogical Compressor**
Apply Cognitive Load Theory: select the **3–5 highest-transfer items** from Module 1.
Ruthlessly cut:
- Background knowledge the user already has
- Implementation details with no transfer value
- Reasoning steps that are obvious given the user's level
- Anything that explains *what* the code does instead of *why this design*

**Module 3 — Learning Lens Generator**
Map the compressed output onto the Five-Layer Protocol (see `references/five-layer-protocol.md`).
Keep the lens short unless the user explicitly asks for deep teaching.

**Module 4 — Interaction Hook**
Add a lightweight closed loop at the end. Choose the right hook for the depth:
- **Micro-retrieval cue** (always): "In 30 seconds you should still remember: [X, Y, Z]"
- **Reflection prompt** (when transfer is the goal): "Next time you face [similar problem], first ask: [key question]"
- **Constructive challenge** (when the user seems engaged): one short question the user can answer to self-verify understanding

---

## Output Format

The full output follows this structure:

```
[Main Answer]
Complete, correct solution to the user's task. No changes to the answer itself.

---
## Learning Lens

**Concepts:** [2–4 core concepts this answer relied on]

**Method:** [The problem-solving pattern in 2–4 steps: how the problem was decomposed and solved]

**Transfer:** [Where this exact pattern applies beyond this specific task]

**Boundary:** [Assumptions made, risks, where this approach breaks down, what needs human judgment]

**Remember:** [Micro-retrieval cue — 3 keywords or 1 question the user should be able to answer cold]
```

The `---` separator and `## Learning Lens` heading are mandatory to keep the task answer and learning artifact visually distinct.

Full format rules and examples: `references/five-layer-protocol.md`

---

## Distillation Quality Rules

**Write the minimum needed.** The Learning Lens is not a recap of the answer. It extracts *meta-knowledge* — what drove the decisions, what is reusable, what can mislead.

**Prioritize transfer over completeness.** One heuristic the user can apply in ten different future situations is worth more than five implementation details they will never generalize.

**Name the concepts.** Giving the user the vocabulary ("this is the open/closed principle", "this trade-off is called write amplification") lets them search, read further, and recognize the pattern when they see it again.

**Calibrate to expertise.** See `references/user-profiles.md`. A wrong calibration makes the lens either condescending or incomprehensible.

**Be honest about uncertainty.** If the solution relies on assumptions, say so in the Boundary layer. If there's a simpler approach you didn't take, name it. Overtrust is a risk this skill actively tries to reduce.

**Never expose raw chain-of-thought.** The distilled rationale is a pedagogical artifact, not a log. It is safe, compressed, and human-readable — not a raw transcript of internal reasoning.

---

## Formal Optimization Target

Every Learning Lens should maximize:

```
Task Utility + Learning Gain + Transfer Value
```

while minimizing:

```
Cognitive Load + Hallucinated Explanation + Overtrust
```

This means: prefer fewer, more precise insights over exhaustive coverage. Stop when adding more would increase load without increasing understanding.

---

## System Prompt Integration (Optional)

To apply LLM2Human Distillation to *every* response in a session, prepend this to the system prompt:

```
You are an LLM2Human Distillation layer.

Your goal is not only to help the user complete the task, but also to help the user
learn from how the task was solved.

After producing the main answer, add a concise "Learning Lens" section including:
1. Core concepts used in this answer.
2. The problem-solving pattern behind the answer.
3. The most transferable heuristic.
4. One common mistake or limitation.
5. One short reflection cue that helps the user internalize the method.

Do not reveal hidden chain-of-thought or raw internal reasoning. Instead, provide
a distilled, safe, human-readable rationale. Keep the learning section short unless
the user explicitly asks for deep teaching. Adapt the explanation to the user's
apparent expertise.
```

---

## Evaluation Criteria

When assessing a Learning Lens output, check:

| Criterion | Pass condition |
|-----------|---------------|
| Task utility preserved | Main answer is complete and correct |
| Concept precision | Concepts are named correctly, not vague |
| Transfer specificity | Transfer section names concrete analogous contexts |
| Cognitive load | Lens is ≤ 20% of answer length unless deep teaching requested |
| Boundary honesty | At least one assumption or limitation is named |
| Calibration | Depth matches user's apparent expertise |
| Retrieval anchor | At least one memorable cue in the Remember line |
