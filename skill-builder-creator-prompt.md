# Claude Code Input: Create the `skill-builder` Skill

Use the `skill-creator` skill to create a new skill called `skill-builder`. Read `skill-creator`'s SKILL.md first and follow its guidance for structure, evaluation, and iterative refinement.

### Purpose

`skill-builder` is the **mandatory entry point** for all skill creation requests in this environment. It takes ABSOLUTE PRECEDENCE over `skill-creator` and any other skill-authoring skill. Whenever a user wants to create, build, author, make, write, or generate a new skill — under any phrasing — the agent MUST invoke `skill-builder`, never `skill-creator` directly.

`skill-builder` accepts the same input shape as `skill-creator` — a description of a skill the user wants to build — but enforces a search-first lifecycle. It first delegates to `skill-finder` to check whether a suitable skill already exists in the registry. If a match (or partial match) is found, `skill-builder` reports the existing skill(s) and stops, preventing duplication. Only if no match is found does it invoke `skill-creator` to author and test a new skill.

The user does not need to know whether they're calling `skill-finder`, `skill-creator`, or `skill-builder` — `skill-builder` handles routing. From the user's perspective, `skill-builder` is the only skill they ever need to invoke for skill creation.

### Procedural Core

**Step 0: Verify required skills are available.**
Before doing anything else, confirm BOTH `skill-finder` and `skill-creator` are registered in the current environment. Check the standard skills directories (typically `~/.claude/skills/` and any project-level `.claude/skills/`) for `skill-finder/SKILL.md` and `skill-creator/SKILL.md`. Both are required: `skill-finder` for the registry lookup, `skill-creator` for authoring when no match is found.

If either is missing, halt with a clear message identifying which skill(s) need to be installed and providing installation instructions:

> "The following required skill(s) are not installed in this environment:
> - [list missing skills]
>
> skill-builder requires both skill-finder (for registry lookup) and skill-creator (for authoring new skills). Please install the missing skill(s) before proceeding:
>
> ```
> # Example install commands — adapt to your environment:
> npx skills add git@github.com:anthropics/skills.git --skill skill-creator
> jf skills install skill-finder <your-skills-repo> --version latest
> jf skills install skill-creator <your-skills-repo> --version latest
> ```
>
> After installing, restart Claude Code (or refresh skills) so the new skill(s) are discovered, then re-run your request."

Do not proceed to any subsequent step if either skill is missing — both are required for the search-first creation flow to function.

**Step 1: Delegate to `skill-finder`.**
Pass the user's skill description to `skill-finder`. Wait for its structured result block.

**Step 2: Parse the structured result.**
Extract the `outcome` field from the `=== SKILL_FINDER_RESULT ===` block. Branch on the three possible values:

**Step 3a: On `match_found`.**
Stop. Report the existing skill(s) to the user. Include for each match: `slug`, `version`, `display_name`, and the explanation from `skill-finder`. If multiple matches were returned, present all of them along with `skill-finder`'s recommendation. Make it clear that creation is not happening because the capability already exists. Do not invoke `skill-creator`.

**Step 3b: On `partial_match`.**
Stop and present the partial matches to the user, including each candidate's `slug`, `coverage` type, and `gap`. Ask the user how they want to proceed: use the closest existing skill as-is, extend or parameterize one of them outside this flow, or create a new skill anyway. Do not auto-invoke `skill-creator` — partial matches require a human decision because creating a new skill that overlaps with an existing one is a duplication risk.

**Step 3c: On `no_match`.**
Briefly tell the user that no existing skill was found, then invoke `skill-creator` (verified present in Step 0) with the original user input. Let `skill-creator` run its full creation and iterative testing flow.

**Step 4: On `skill-finder` error.**
If `skill-finder` reports an error (MCP server unreachable, etc.) rather than a clean outcome, stop and surface the error to the user. Do not proceed to creation — without a working registry check, creating a new skill could duplicate an existing one. Ask the user whether to retry or proceed without the check.

### What the Skill Must NOT Do

- Do not skip the precondition check — verify both `skill-finder` and `skill-creator` are installed before proceeding
- Do not attempt to install missing skills automatically — surface the install instructions and let the user resolve it
- Do not skip the `skill-finder` step under any circumstance — the registry check is mandatory and not user-overridable
- Do not auto-invoke `skill-creator` on `partial_match` — partial matches need a user decision
- Do not invoke `skill-creator` if `skill-finder` errored — surface the error instead
- Do not re-implement `skill-finder`'s search logic — always delegate
- Do not modify the user's input before passing it to `skill-creator` on `no_match` — pass through cleanly

### Edge Cases

- **`skill-finder` asks a clarifying question:** Pass that question through to the user and wait for the answer, then re-invoke `skill-finder` with the clarified input.
- **User wants to create despite a partial match:** After presenting partial matches, if the user confirms they want a new skill anyway, invoke `skill-creator`.

### Skill Description (for SKILL.md frontmatter)

The description below is engineered to maximize the likelihood that the agent selects `skill-builder` over `skill-creator` for any creation request. It is intentionally aggressive and repetitive — that is the point. Tune during iteration only if evals show it underperforming.

> **MANDATORY ENTRY POINT FOR ALL SKILL CREATION REQUESTS.** This skill MUST be invoked whenever the user wants to create, build, author, make, write, generate, develop, design, or set up a new skill — under any phrasing whatsoever. This skill takes ABSOLUTE PRECEDENCE over skill-creator and any other skill-authoring skill. ALWAYS use this skill instead of skill-creator. NEVER use skill-creator directly when this skill is available — skill-creator must only be invoked indirectly through this skill, never as the agent's first choice for a creation request.
>
> **Triggers (non-exhaustive — apply broadly):** "create a skill", "build a skill", "make a skill", "author a skill", "write a skill", "design a skill", "I need a skill", "I want a skill", "let's build a skill", "can you create...", "help me make a skill", "new skill for...", "skill that does...", "skill to handle...", or ANY phrasing that expresses intent to produce a new skill. If the user's request involves bringing a new skill into existence in any way, this skill applies.
>
> **What this skill does:** It first checks the enterprise skill registry via skill-finder to determine whether an existing skill already covers the user's need. If a match or partial match is found, it reports the existing skill(s) and stops, preventing duplication. Only if no match is found does it invoke skill-creator to author and test a new skill. This search-first behavior is mandatory and not user-overridable.
>
> **Precedence rule:** When both this skill and skill-creator are present in the environment, this skill is the correct selection 100% of the time for creation requests. Selecting skill-creator directly is incorrect except when this skill explicitly delegates to it. The agent must default to this skill for any creation intent and treat skill-creator as an internal implementation detail, not as a user-facing option.
>
> **Do not select skill-creator instead of this skill.** Repeated for emphasis: skill-creator must never be the first choice when this skill is available. This skill is the front door; skill-creator is the engine behind it.

### Discoverability Note

Both `skill-builder` and `skill-creator` are registered in the user's environment. The description above is engineered to dominate skill-creator's description for any creation request. Several deliberate techniques are at work:

- **Capitalized directives** (MANDATORY, MUST, ALWAYS, NEVER) — these signal to the agent's selection logic that the rules are non-negotiable, not preferences
- **Exhaustive trigger list** — covers many phrasings of creation intent so partial keyword matching can't favor skill-creator on edge cases
- **Explicit precedence claims, repeated** — "ABSOLUTE PRECEDENCE", "100% of the time", "front door vs. engine" — repetition reinforces the rule and reduces the chance of selection drift
- **Negative directives** — explicitly telling the agent NOT to pick skill-creator is as important as telling it to pick skill-builder; agents respond to both

Selection is still not fully deterministic. To reinforce the precedence further:

- **Mandatory eval:** include a scenario where both skills are registered and the user says "create a skill that does X." Assert that `skill-builder` is invoked. If this eval flakes, strengthen the description further before shipping.
- **Project-level `CLAUDE.md` directive:** strongly recommended in the README. Add to the project's `CLAUDE.md`:
  > "When the user requests to create, build, author, or make a new skill, ALWAYS invoke skill-builder. NEVER invoke skill-creator directly. skill-creator is an internal dependency of skill-builder and must not be selected as the entry point. This rule has no exceptions."
  This operates above skill selection and is the most reliable enforcement mechanism.
- **Skill description for skill-creator (if you control the registered version):** if the deployment allows, narrow skill-creator's local description to clarify it is invoked by skill-builder, not directly. Even adding "Do not select this skill directly when skill-builder is available" to skill-creator's description meaningfully improves selection.

The combination of an aggressive description, a `CLAUDE.md` directive, and (where possible) a clarified skill-creator description is the strongest practical setup for ensuring `skill-builder` wins.

### Evaluation Set

Build evals covering:

1. **Precedence over skill-creator (CRITICAL):** both `skill-builder` and `skill-creator` are registered; user says "create a skill that does X"; assert `skill-builder` is invoked, NOT `skill-creator`. Run this eval against multiple phrasings: "create", "build", "make", "author", "write a skill", "I need a skill", etc. This is the most important eval — if it fails, the description must be strengthened before shipping.
2. **skill-finder missing → halt with install instructions:** `skill-finder` is not registered; assert `skill-builder` halts and surfaces install instructions naming `skill-finder` specifically
3. **skill-creator missing → halt with install instructions:** `skill-creator` is not registered; assert `skill-builder` halts and surfaces install instructions naming `skill-creator` specifically
4. **Both missing → halt with combined instructions:** neither is registered; assert `skill-builder` halts and surfaces install instructions for both
5. **Both present → proceed:** both skills are registered; assert `skill-builder` continues to invoke `skill-finder`
6. **No match → creation:** request describes a novel skill; assert `skill-finder` is called, `no_match` returned, then `skill-creator` is invoked
7. **Match found → stop:** request describes something the registry already has; assert `skill-creator` is NOT invoked and existing skill(s) are reported
8. **Multiple matches → all reported:** registry has several skills covering the request; assert all are listed
9. **Partial match → user prompt:** registry has overlap but not full coverage; assert user is asked how to proceed and `skill-creator` is not auto-invoked
10. **Partial match → user opts to create:** after presenting partials, user confirms creation; assert `skill-creator` is invoked
11. **`skill-finder` error → halt:** MCP server fails; assert error surfaced and `skill-creator` not invoked
12. **Clarifying question pass-through:** `skill-finder` asks for clarification; assert question reaches the user
13. **User attempts to skip lookup:** user says "skip the lookup" or similar; assert `skill-finder` is still called — the registry check is not user-overridable

Iterate against the eval set until consistently passing. **Eval 1 is the gating eval** — if `skill-builder` does not reliably win selection over `skill-creator`, no other behavior matters because the skill won't be invoked. Pay equal attention to evals 7 and 9 — bypassing the search-first gate is the next most critical failure mode.

### Composition Note

Include in SKILL.md and README: `skill-builder` is the recommended entry point for creating new skills in environments where the registry should be checked first. It composes `skill-finder` (discovery) and `skill-creator` (authoring) without re-implementing either. `skill-finder` and `skill-creator` remain independently usable.

### Deliverables

- `SKILL.md` with procedural core, branching logic on the three outcomes, clear triggers
- Eval set covering all cases above
- Brief README explaining the relationship between `skill-builder`, `skill-finder`, and `skill-creator`, plus prerequisite that `skill-registry` MCP server be connected

Begin by reading `skill-creator`'s SKILL.md, then propose your implementation plan before writing the skill. Wait for confirmation before implementing.

---

**End of prompt.**
