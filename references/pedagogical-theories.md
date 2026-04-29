# Pedagogical Theories — LLM2Human Distillation

This document grounds the LLM2Human skill in six learning-science frameworks. Each section states what the theory implies for how an AI agent should structure its output.

---

## A. Cognitive Apprenticeship — Making Thinking Visible

**Core idea:** Expert cognition is normally invisible. Novices learn by watching experts work, but only if the expert externalizes their reasoning. Collins, Brown & Newman (1989) formalized four instructional mechanisms: **modeling** (showing how), **coaching** (guiding while doing), **scaffolding** (providing support structures), and **fading** (removing scaffolds as competence grows).

**Implication for LLM2Human:**
The agent should not just deliver the final answer. After every non-trivial task, it should model the expert perspective that produced that answer: how the problem was decomposed, what knowledge was invoked, what the key decision points were, and how the user could reproduce this reasoning on a similar problem.

The agent acts as the expert making their thinking visible — not explaining the answer in terms of itself, but explaining the *process* that generated it.

**Failure mode to avoid:** Showing all intermediate steps without compression. Raw chain-of-thought is not modeling — it is a transcript. Effective modeling highlights the moments that distinguish novice from expert reasoning, not every intermediate thought.

---

## B. Cognitive Load Theory — Avoid Infinite Explanation

**Core idea:** Sweller (1988) and Mayer's Cognitive Theory of Multimedia Learning both posit that human working memory has limited capacity. Learning requires selecting, organizing, and integrating information. When extraneous cognitive load is high (irrelevant material, redundancy, poor organization), learning is suppressed even if the information is technically correct.

Three types of cognitive load:
- **Intrinsic:** inherent complexity of the material (unavoidable)
- **Extraneous:** caused by poor presentation (avoidable — our target to minimize)
- **Germane:** load that builds schemas (should be optimized)

**Implication for LLM2Human:**
The Learning Lens must be *selective*, not comprehensive. The minimum useful distillation is 3–5 transferable insights. Anything beyond that increases extraneous load without adding germane load. The skill should ruthlessly cut:
- Background knowledge already established by the conversation
- Implementation details with no generalization value
- Correct-but-obvious reasoning the user already understands

**Practical rule:** The Learning Lens should be ≤ 20% of the answer length by default. If the user asks for deep teaching, this limit relaxes — but every addition must earn its cognitive cost.

---

## C. Worked Examples + Self-Explanation — Schemas Over Steps

**Core idea:** Sweller & Cooper (1985) showed that novice learners acquire problem-solving schemas faster from studying worked examples than from solving equivalent problems independently. Chi et al.'s (1989) research on **self-explanation** extended this: learners who actively explained worked examples to themselves — connecting steps to underlying principles — outperformed those who merely read them.

The key is not just showing *what* was done, but surfacing *why each step was taken* and *which principle it instantiates*.

**Implication for LLM2Human:**
Every Learning Lens is a worked example of the task's solution. The **Method** layer shows the solving sequence. But crucially, the **Concept** layer names the underlying principles each step instantiated. This combination enables self-explanation: the user can link the steps they see to the named principles they can look up, internalize, and reapply.

**Example contrast:**

*Without self-explanation support:*
> "I used binary search to find the value."

*With self-explanation support (LLM2Human style):*
> "Binary search applies here because the array is sorted and the problem asks for a specific value. The invariant is: at each step, the target is either in the left half or the right half. This is the divide-and-conquer principle — each decision halves the remaining search space, giving O(log n) rather than O(n)."

---

## D. ICAP Framework — From Passive to Constructive Learning

**Core idea:** Chi & Wylie (2014) categorized learning engagement into four modes:
- **Passive:** observing, reading, listening (no overt behavior)
- **Active:** highlighting, underlining, copying (surface manipulation)
- **Constructive:** generating explanations, making comparisons, drawing diagrams (going beyond given information)
- **Interactive:** dialoguing, debating, co-explaining with others

Learning effectiveness follows the order: **Interactive > Constructive > Active > Passive.**

**Implication for LLM2Human:**
If the skill only outputs a Learning Lens that the user reads, it produces Passive engagement. To push toward Constructive, the skill must add an **Interaction Hook** — a prompt that asks the user to generate something:
- Summarize the core trade-off in one sentence
- Predict what would happen if a key constraint changed
- Name one other context where this pattern applies

This lightweight generative prompt moves the user from reading to constructing. It does not require the user to respond — even the act of attempting to answer internally increases engagement to Active or Constructive mode.

**Practical rule:** Every Learning Lens includes at least a Retrieval Anchor (passive→active) and optionally a Constructive Challenge for more engaged learners.

---

## E. Retrieval Practice — Testing Enhances Retention

**Core idea:** Roediger & Karpicke (2006) demonstrated that testing memory — actively retrieving information — produces more durable long-term retention than additional study of the same material. This is the **testing effect** or **retrieval practice effect**. Active recall, even imperfect, strengthens memory traces more than re-exposure.

Key nuance: retrieval practice works even when the learner *cannot fully recall* the answer. The effort of retrieval is beneficial independent of success.

**Implication for LLM2Human:**
The **Remember** line at the end of every Learning Lens is a micro-retrieval cue. It gives the user:
1. A 3-keyword anchor — three concepts to try to recall without looking
2. Or a single diagnostic question — something they should be able to answer cold after the task

This is not a quiz; it is a mnemonic framing. The user is primed to attempt retrieval 30 seconds, 30 minutes, and 3 days later. Even passive re-reading of the keywords triggers partial retrieval and strengthens the memory trace.

**Practical rule:** The Remember line should be concrete and specific enough that the user can genuinely attempt recall. "Understand the algorithm" is too vague. "Remember: monotonic stack, O(n) amortized, each element pushed and popped at most once" is a retrieval target.

---

## F. XAI and Human-AI Mental Models — Calibrated Trust

**Core idea:** Research in Explainable AI (Doshi-Velez & Kim 2017) and Human-AI interaction (Parasuraman & Riley 1997) shows that the purpose of AI explanation is not explanation per se, but helping users form a **correct mental model** of the AI's capabilities and limitations. Incorrect mental models lead to:
- **Over-reliance:** accepting AI outputs without appropriate scrutiny
- **Under-reliance:** ignoring useful AI guidance due to distrust
- **Miscalibrated trust:** trusting in the wrong domains and distrusting in the right ones

**Implication for LLM2Human:**
The **Boundary** layer is specifically designed to combat over-reliance. It must answer:
- What assumptions does this conclusion rely on?
- Where might this approach fail or produce wrong results?
- Which parts of this answer require human judgment or domain verification?

The goal is not to undermine the answer but to help the user apply it correctly. A user who knows "this approach assumes the dataset fits in memory" will generalize the technique correctly. A user who doesn't will misapply it in production.

**Failure mode to avoid:** Hedging everything with "results may vary." That is epistemic noise, not calibration. The Boundary layer should name *specific* assumptions and *specific* failure modes — not generic disclaimers.

---

## Integration: How the Six Theories Map to the Five Layers

| Layer | Primary Theory | What It Does |
|-------|---------------|--------------|
| Answer | — | Task utility; no theory needed, just correctness |
| Concept | Cognitive Apprenticeship, Worked Examples | Names the expert knowledge invoked |
| Method | Cognitive Apprenticeship, Worked Examples | Shows the problem-solving sequence with principle links |
| Transfer | ICAP, Retrieval Practice | Moves the user toward constructive engagement; builds schemas |
| Boundary | XAI / Mental Models | Calibrates trust; prevents misapplication |
| Remember | Retrieval Practice | Micro-retrieval cue for long-term retention |

Cognitive Load Theory is a constraint on *all* layers: keep each layer minimal enough to be absorbed, not comprehensive enough to overwhelm.
