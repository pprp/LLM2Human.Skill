# LLM2Human Distillation Skill

Transforms any agent task solution into a human-learnable artifact. Instead of just delivering an answer, the agent appends a **Learning Lens** — a compact, pedagogically structured section that helps you understand *how* the problem was solved so you can reproduce the reasoning yourself.

Based on six learning-science principles: Cognitive Apprenticeship, Cognitive Load Theory, Worked Examples, ICAP Framework, Retrieval Practice, and XAI Mental Model calibration.

---

## What It Produces

After every non-trivial task, the skill appends:

```
---
## Learning Lens

**Concepts:**   2–4 named concepts the solution relied on

**Method:**     The problem-solving pattern in 3–5 steps

**Transfer:**   Where else this exact reasoning applies

**Boundary:**   Assumptions, risks, and failure modes

**Remember:**   3 keywords or 1 diagnostic question for self-testing
```

Depth adapts automatically to your expertise level based on conversational signals.

---

## Installation

### Claude Code (CLI / IDE extension)

```bash
# Clone this repository
git clone https://github.com/YOUR_USERNAME/LLM2Human.Skill.git

# Copy the skill into your Claude skills directory
cp -r LLM2Human.Skill ~/.claude/skills/llm2human
```

That's it. The skill is immediately available in any Claude Code session.

**Verify installation:**
```bash
ls ~/.claude/skills/llm2human/
# Should show: SKILL.md  references/
```

### Claude Code via npx (if published to npm)

```bash
npx skills add YOUR_USERNAME/LLM2Human.Skill
```

### Manual install (single file)

If you only want the skill definition without the reference docs:

```bash
mkdir -p ~/.claude/skills/llm2human
curl -o ~/.claude/skills/llm2human/SKILL.md \
  https://raw.githubusercontent.com/YOUR_USERNAME/LLM2Human.Skill/main/SKILL.md
```

---

## Usage in Claude Code

### Invoke on demand

After the skill is installed, trigger it by saying:

```
/llm2human
```

Or use natural phrases — the skill auto-triggers on:

| Phrase | Language |
|--------|----------|
| "teach me how you did that" | EN |
| "explain your thinking" | EN |
| "I want to learn from this" | EN |
| "walk me through your reasoning" | EN |
| "learning mode" | EN |
| "what concepts are at play here" | EN |
| "how would I do this myself next time" | EN |
| "我想学习" / "教我" / "解释思路" | ZH |

### Always-on mode (project level)

To apply LLM2Human Distillation to every response in a project, add this block to your `CLAUDE.md`:

```markdown
## Learning Mode

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

### Skip the lens for a single response

```
[your question here] — skip the learning lens this time
```

---

## Using PROMPT.md — Universal Memory File

`PROMPT.md` is a single self-contained file that encodes the full LLM2Human Distillation behavior. Paste it into any LLM's memory or system prompt to activate the skill without installing anything.

### ChatGPT (GPT-4o / GPT-4)

1. Open ChatGPT → click your profile → **Settings**
2. Go to **Personalization → Memory**
3. Click **Manage memories → Add a memory**
4. Paste the full contents of `PROMPT.md`

Or use it as a system prompt in the API:

```python
import openai, pathlib

system_prompt = pathlib.Path("PROMPT.md").read_text()

client = openai.OpenAI()
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": "How does binary search work?"},
    ],
)
print(response.choices[0].message.content)
```

### Google Gemini

**Gemini Advanced (web):**
1. Start a new conversation
2. Click the **System instructions** field (gear icon or "+" in some interfaces)
3. Paste the contents of `PROMPT.md`

**Gemini API:**

```python
import google.generativeai as genai
import pathlib

system_prompt = pathlib.Path("PROMPT.md").read_text()

genai.configure(api_key="YOUR_API_KEY")
model = genai.GenerativeModel(
    model_name="gemini-1.5-pro",
    system_instruction=system_prompt,
)

response = model.generate_content("How does binary search work?")
print(response.text)
```

### Any other LLM or chat interface

Paste `PROMPT.md` as the first message in your conversation, prefixed with:

```
Use the following as your operating instructions for this session:

[paste PROMPT.md contents here]
```

### Skip the lens for a single response

```
[your question here] — skip the learning lens this time
```

---

## Installation in OpenAI Codex

Codex does not have a native skill system, but you can apply LLM2Human Distillation as a **system prompt** in any Codex-powered tool (Codex CLI, API calls, or custom integrations).

### Codex CLI (system prompt file)

If you are using the [OpenAI Codex CLI](https://github.com/openai/codex), create or edit your system prompt file:

```bash
# Create a system prompt file
cat > ~/.codex/system-prompt.md << 'EOF'
You are an LLM2Human Distillation layer.

Your goal is not only to help the user complete the task, but also to help the user
learn from how the task was solved.

After producing the main answer, add a concise "Learning Lens" section including:
1. Core concepts used in this answer (2–4 named concepts).
2. The problem-solving pattern in 3–5 steps.
3. The most transferable heuristic — where else this pattern applies.
4. One common mistake, assumption, or limitation.
5. One short reflection cue: either 3 keywords to remember, or one diagnostic
   question the user can answer cold to verify they understood.

Format the Learning Lens as:
---
## Learning Lens
**Concepts:** ...
**Method:** ...
**Transfer:** ...
**Boundary:** ...
**Remember:** ...

Do not reveal raw internal reasoning. Provide a distilled, human-readable rationale.
Keep the learning section short (≤ 20% of answer length) unless the user asks for
deep teaching. Adapt depth to the user's apparent expertise level.
EOF
```

Then reference it when running Codex:

```bash
codex --system-prompt ~/.codex/system-prompt.md "your question here"
```

### Codex API (Python)

```python
import openai

LEARNING_LENS_SYSTEM_PROMPT = """
You are an LLM2Human Distillation layer.

After producing the main answer to any task, add a "Learning Lens" section:

**Concepts:** 2–4 named concepts the solution relied on
**Method:** The problem-solving pattern in 3–5 steps (what + why for each step)
**Transfer:** 2–3 contexts where this exact pattern applies beyond this task
**Boundary:** Assumptions made, risks, and where this approach breaks down
**Remember:** 3 keywords OR one diagnostic question the user can self-test with

Rules:
- Keep Learning Lens to ≤ 20% of answer length unless deep teaching is requested
- Name concepts explicitly — never describe without naming
- Boundary must have specific assumptions, not generic disclaimers
- Adapt depth to the user's apparent expertise (novice → define; expert → name only)
- Never expose raw chain-of-thought; produce distilled, safe rationale only
""".strip()

client = openai.OpenAI()

def ask_with_learning_lens(question: str) -> str:
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": LEARNING_LENS_SYSTEM_PROMPT},
            {"role": "user", "content": question},
        ],
    )
    return response.choices[0].message.content

# Example
print(ask_with_learning_lens("How does binary search work?"))
```

### Codex API (Node.js / TypeScript)

```typescript
import OpenAI from "openai";

const LEARNING_LENS_SYSTEM_PROMPT = `
You are an LLM2Human Distillation layer.

After producing the main answer to any task, add a "Learning Lens" section:

**Concepts:** 2–4 named concepts the solution relied on
**Method:** The problem-solving pattern in 3–5 steps (what + why for each step)
**Transfer:** 2–3 contexts where this exact pattern applies beyond this task
**Boundary:** Assumptions made, risks, and where this approach breaks down
**Remember:** 3 keywords OR one diagnostic question the user can self-test with

Keep Learning Lens to ≤ 20% of answer length. Adapt depth to user expertise.
`.trim();

const client = new OpenAI();

async function askWithLearningLens(question: string): Promise<string> {
  const response = await client.chat.completions.create({
    model: "gpt-4o",
    messages: [
      { role: "system", content: LEARNING_LENS_SYSTEM_PROMPT },
      { role: "user", content: question },
    ],
  });
  return response.choices[0].message.content ?? "";
}
```

---

## Example Output

**User:** How do I find the second-highest salary in SQL?

**Answer:**
```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

---
## Learning Lens

**Concepts:**
- **Scalar subquery** — a SELECT inside another SELECT that returns one value; used here to compute the ceiling to exclude
- **Exclusion-then-maximize** — exclude the known maximum, then find the new maximum; the standard pattern for "nth largest" problems
- **Non-correlated subquery** — runs once (not once per row), so cost is O(n) not O(n²)

**Method:**
1. Recognize "second highest" as "highest of everything below the highest" → maps to exclusion-then-maximize
2. Inner query computes the ceiling: `SELECT MAX(salary)`
3. Outer query filters below it and re-maximizes: `WHERE salary < [ceiling]`
4. Check edge cases: equal salaries return one value; single-row table returns NULL

**Transfer:**
- Nth largest/smallest in any ordered column — nest one more exclusion layer, or use `OFFSET n-1`
- "Best result excluding the known winner" in any analytics query
- Filtering out known extremes before aggregation in reporting pipelines

**Boundary:**
- Tied second-highest: returns one value even if multiple employees share it — use `DENSE_RANK()` if you need all tied rows
- NULL salaries: `MAX()` silently ignores NULLs — verify this matches business intent
- Scale: double full-scan is fine up to ~1M rows; window functions are faster with an index at scale

**Remember:** exclusion-then-maximize, non-correlated scalar subquery, NULL and tie edge cases

---

## File Structure

```
LLM2Human.Skill/
├── SKILL.md                          # Main skill definition (Claude Code)
├── README.md                         # This file
└── references/
    ├── pedagogical-theories.md       # Six learning-science theories with agent implications
    ├── five-layer-protocol.md        # Output format spec with annotated examples
    └── user-profiles.md              # Novice / practitioner / expert calibration rules
```

---

## Design Principles

**Distillation, not transcription.** The Learning Lens is not a replay of the agent's reasoning steps. It extracts the meta-knowledge — what drove the decisions, what is reusable, what can mislead.

**Minimum viable insight.** 3–5 transferable points beat a comprehensive tour. Cognitive Load Theory: every item in the lens must earn its cognitive cost.

**Transfer over detail.** One heuristic applicable to ten future situations is worth more than five implementation specifics that never generalize.

**Calibrated trust.** The Boundary layer exists to reduce overtrust. It names specific assumptions and failure modes — not generic disclaimers.

**Retrieval over re-reading.** The Remember line is a retrieval-practice cue, designed to be tested cold. "Understand the algorithm" is not a retrieval target. "Monotonic stack, amortized O(n) by push/pop symmetry, invariant as correctness argument" is.

---

## References

- Collins, Brown & Newman (1989) — Cognitive Apprenticeship
- Sweller (1988), Mayer (2001) — Cognitive Load Theory / Multimedia Learning
- Sweller & Cooper (1985), Chi et al. (1989) — Worked Examples and Self-Explanation
- Chi & Wylie (2014) — ICAP Framework
- Roediger & Karpicke (2006) — Testing Effect / Retrieval Practice
- Doshi-Velez & Kim (2017) — Explainable AI and Mental Models
- Hinton et al. (2015) — Knowledge Distillation (inspiration for the distillation framing)
