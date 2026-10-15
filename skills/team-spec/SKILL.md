---
name: team-spec
description: Build a specification from supplied context using a team of sub-agents with different personas (engineer, tester, QA, researcher, security, etc.), then have an arbiter decide what goes in. Follows the project's existing spec format if one is defined, otherwise writes when-then / given-when-then (BDD) scenarios. Use when the user says "team-spec", "write a spec for X", "spec this out from every angle", or wants a rigorous, over-constrained specification rather than a single-pass draft.
---

# team-spec

Turn a description of *what is being defined* into a specification. Many personas contribute; one arbiter decides. Bias: **an over-constrained spec beats an under-constrained one.**

## Step 1 — Find the spec format

Before anything else, check whether the working context already defines a specification format:

- `CLAUDE.md`, `AGENTS.md`, `README`, `CONTRIBUTING`, `docs/`, `spec/`, `specs/`, `rfcs/`, `adr/`, templates (`*template*`), and a few existing spec files.
- Anything the user said in this conversation.

Decide one of:

1. **Defined format found** → use it exactly (sections, numbering, IDs, keywords like MUST/SHOULD, file location and naming). Note the source path.
2. **None found** → use BDD: `Given <precondition> / When <event> / Then <observable outcome>` (use plain `When / Then` when there is no meaningful precondition). Group scenarios under headed features, give each a stable ID (`FEAT-01.S3`).

State the chosen format and its source in one line, then pass it verbatim to every sub-agent.

## Step 2 — Get the context

The user invokes the skill with the path to a **context markdown file** (e.g. `/team-spec research/foo.md`). That file holds the background research and the focus of the work, and the spec is derived from it.

- Read the whole file. If no path was given or it can't be read, ask once; do not proceed from memory or guesswork.
- Follow links or file references inside it only where they are local and clearly relevant.
- Separate what it contains into: **focus** (what is being specified), **background/research** (facts, findings, prior art), **constraints and non-goals**. Treat research as evidence, not as requirements: a finding becomes a spec entry only if it bears on the focus.
- Do not invent facts beyond the file. Gaps and contradictions in it go to Open questions.
- Hand every sub-agent the **file path** and the full file contents, plus your focus/constraints summary, so they work from the source and not only your paraphrase. Name the file path in the spec's header as its source.

## Step 3 — Contributors (parallel)

Spawn all contributors **in a single message** so they run concurrently. Give each: the context file (path and contents) with your summary, the format from Step 1, its persona, and these rules:

- Contribute only what your persona would insist on; do not try to cover everything.
- Each item is a spec entry **in the required format**, plus a one-line rationale and a tag `[persona]`.
- Items must be concrete and testable: specific inputs, thresholds, states, error behaviours. No vague wording ("fast", "secure", "user-friendly").
- Cover negative paths, boundaries and failure modes, not just the happy path.
- Flag assumptions and open questions separately; never silently resolve ambiguity.

### Choosing personas

Use at least 5, distinct in angle. Pick from this roster, drop what doesn't apply, add domain-specific ones (e.g. DBA, data engineer, accessibility specialist, compliance officer):

| Persona | Pushes for |
|---|---|
| Engineer | Feasibility, interfaces, data contracts, state, ordering, concurrency, error handling |
| Researcher | Prior art, standards, precedent, how comparable systems behave, terminology accuracy |
| Tester | Verifiable assertions, boundaries, equivalence classes, determinism, observability for tests |
| QA | End-to-end flows, regressions, real-world misuse, environments, acceptance criteria |
| Product / End user | Intent, user-visible behaviour, defaults, messages, non-goals |
| Security / Adversary | Authn/z, input abuse, data exposure, abuse cases, "what if someone tries to break it" |
| SRE / Operator | Failure, recovery, limits, timeouts, monitoring, upgrade/rollback, config |
| Performance | Latency/throughput/memory targets, scale limits, degradation behaviour |
| Compatibility | Backwards/forwards compatibility, migration, rolling upgrades, versioning |
| Skeptic | Contradictions, ambiguities, gaps between other contributions, unstated assumptions |

**Hard minimum:** the team must always include a Tester and at least one adversarial contributor (Security/Adversary or Skeptic), whatever the topic. Never spawn the team without one. An adversary's job is to attack: abuse cases, hostile or malformed inputs, contradictions and gaps in the context file and in the other contributions, and "how does this fail or get exploited". If the topic isn't security-related, use the Skeptic or frame the Adversary around misuse and failure rather than dropping it. The spec's value is in the cases people forget.

The arbiter must not discard an adversarial contribution wholesale; each of its items is judged on its merits and, when in doubt, kept.

## Step 4 — Arbiter

Spawn one arbiter sub-agent. Give it the context file, the format, and every contribution verbatim. It decides what enters the specification:

- **Include** by default. When in doubt, include; a constraint that proves unnecessary is cheap to remove, a missing one is expensive to discover.
- **Dismiss** only items that are: duplicates (merge into the best-worded one, keeping the strictest threshold), contradicted by the brief, out of scope, untestable/vague with no way to sharpen, or pure speculation unrelated to the stated context.
- **Resolve conflicts** between contributors by choosing the stricter behaviour, or by recording both under an explicit open question if the brief can't decide.
- **Sharpen** weak-but-valid items so they are concrete and testable.
- **Consult** when an include/dismiss call hinges on missing information (unclear rationale, possible conflict, doubtful testability). The arbiter may put questions to the contributing team members before ruling. Keep each contributor's agent ID from Step 3. The arbiter returns a list of `{persona, question, item ID}` instead of a ruling; you relay each question to that contributor with `SendMessage` (all in parallel), then resume the arbiter (`SendMessage`, or a fresh arbiter given the full history) with the answers. Repeat as needed, at most 3 rounds, then the arbiter must rule. Answers are evidence, not votes: the arbiter still decides, and unresolved doubt defaults to include.
- Produce: (a) the final spec in the required format with stable IDs, (b) a **Dismissed** table (item, persona, reason) , (c) **Open questions / assumptions** for the user.

## Step 5 — Deliver

Write the spec to the location the project's format dictates; otherwise `specs/<kebab-name>.md`. Append the Dismissed table and Open questions as trailing sections (or a sibling `<name>.review.md` if the project's format forbids extras). Report to the user: file path, format used and why, counts (contributed / included / dismissed), and the open questions. Do not paste the whole spec into chat.
