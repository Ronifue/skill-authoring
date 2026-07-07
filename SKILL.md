---
name: skill-authoring
description: >
  Load when the user asks to "create a skill", "write a SKILL.md",
  "review this skill", "check my skill's quality", or
  "why isn't my skill triggering". Covers trigger descriptions, routing,
  common patterns, checklists, anti-patterns, and maintenance.
  Do NOT use for editing skill wording — load prompt-governance.
  Do NOT use for one-time prompts or AGENTS.md edits.
---

# Skill Authoring

Guide for writing skills that trigger reliably and survive long sessions.

## Task Routing

| Your task | Start here |
|---|---|---|
| "create a skill" / "write a SKILL.md" | Quickstart → Skill Structure → Common Patterns |
| "review this skill" / "check quality" | Quality Checklist |
| "should I create a skill for this?" | When to Create → Decision Tree |
| "why isn't my skill triggering" | §Description anatomy (in Skill Structure) |
| "debug / why does it behave wrong" | Core Design Principles |
| "update a skill I wrote months ago" | Anti-Drift |
| "audit before installing" | Auditing |

---

## Quickstart

A complete minimal skill file. The rest of this document explains each part.

```yaml
---
name: example-skill
description: >
  Load when the user asks to "generate a report" or "export data".
  Do not use for dashboard config.
---
```

```markdown
# Example Skill

## Always Read
- references/data-schema.md (output column names)

## Common Tasks
| Task | Required reads |
|---|---|
| Report | references/templates.md → check §required-fields |
| Export | references/formats.md → run scripts/validate.py |
| Other | Scan full SKILL.md |

## Known Gotchas
- Date fields are UTC — references/gotchas.md#date-utc

## Report Checklist
- [ ] Schema fields verified
- [ ] Template matches report type
- [ ] Output validated: `scripts/validate.py --report <file>`
```

---

## Skill Structure

### Directory Layout

```
skill-name/
├── SKILL.md         ← required: description (trigger) + body (router)
├── references/      ← long docs, schemas (loaded per task). Name files by content, not number: form_validation_rules.md not doc2.md.
├── scripts/         ← repeatable logic
├── assets/          ← templates, fonts, images
└── agents/          ← Codex: openai.yaml for UI metadata + invocation policy

**Paths:** Always use `/` (forward slashes), even on Windows. Backslashes break cross-platform execution.  
**No extras:** No README.md or CHANGELOG.md — agent reads all files, human docs waste context budget.

### Degrees of Freedom — Choosing the Right Shape

Match specificity to task fragility. This framework decides which of the three shapes below to use:

| Freedom | When | Shape |
|---|---|---|
| **High** (text instructions) | Multiple approaches valid, context-dependent, heuristics guide | Pure reference |
| **Medium** (scripts with params) | Preferred pattern exists, some variation OK, config affects behavior | Inline-shell |
| **Low** (exact scripts) | Operations fragile and error-prone, consistency critical, fixed sequence | CLI-first |

**Check:** Find the most fragile step in this task. What freedom level does it need? That picks your shape.

### Three Skill Shapes

| Shape | When | Example |
|---|---|---|
| **CLI-first** | Logic >3 lines | PDF processing via Python |
| **Inline-shell** | 1-2 trivial commands | `npx validate .` |
| **Pure reference** | Templates, taxonomies | Brand colors, error codes |

### SKILL.md Anatomy

**name**: Lowercase + hyphens, matches directory name. Gerund form (`pdf-processing` not `PdfProcessor`).

**description**: Routing trigger, not a summary. Write "Load when the user asks to…" not "This skill does…". Target ~500 chars / ~100 tokens. Include user keywords and anti-triggers.

**Why tight matters:** Descriptions have an independent budget (~8K chars Claude Code, ~5.4K Codex). Over-budget descriptions are silently truncated — the trigger vanishes. Explicit invocation (`/skills`) bypasses this; implicit (description match) cannot.

```yaml
# Wrong: summary
description: Helps with API tasks.
# Right: trigger with anti-trigger
description: >
  Load when the user asks to "add an endpoint" or "write a controller".
  Do not use for frontend changes.
```

**Body**: Router, not encyclopedia. This skill's own sections (Task Routing, Quickstart, Skill Structure, Common Patterns, Core Design Principles) are one valid shape. For skills with references/ and scripts/, a production body often uses:

```
## Always Read        ← 2-3 files every task reads
## Common Tasks       ← task-type → files + workflow (5-10 + "Other" fallback)
## Known Gotchas      ← one-liner + anchor to reference
```

The pattern matters more than the labels — every section should answer "what task do I help with?"

Anti-trigger test: list 3 similar requests that should NOT trigger. If you can't, description is too broad.

---

## Common Patterns

Start with the Gotchas pattern — every skill needs it. Add Template, Checklist, or Validation Loop only when the task demands their specific structure.

### Gotchas Pattern — for known recurring failures
One-liner in SKILL.md → detailed fix in references/.

```markdown
## Known Gotchas
- Filter registers before app init or first render is blank
  → references/gotchas.md#filter-registration
- Forward slashes required in paths, even on Windows
  → references/gotchas.md#cross-platform-paths
```

Record only those meeting ≥2/3 of: {repeatable, high-cost, code-invisible}.

### Template Pattern — for outputs matching a specific format

```markdown
## Commit Format
Output exactly:
```
<type>(<scope>): <subject>
```
type ∈ {feat, fix, refactor, docs}, scope optional, subject ≤ 72 chars.
```

### Checklist Pattern — for multi-step workflows
Copyable checkbox list the agent can check off.

```markdown
## Pre-Release
- [ ] Tests pass: `npm test`
- [ ] No TS errors: `npx tsc --noEmit`
- [ ] CHANGELOG entry matches conventional commits
```

### Validation Loop Pattern — for objective pass/fail
Specify: validator → expected → fix → repeat.

```markdown
## Build Verification
1. Run `npm run build`. Exit 0? Proceed.
2. If errors: fix first error, go to step 1.
3. Max 3 loops. After 3rd failure: report and stop.
```

---

## Core Design Principles

Each principle has a Check question. An anti-pattern reminder follows where the model commonly backslides.

### 1. Delta Principle
Only add what the model gets wrong or doesn't know.
**Check:** Would the model answer correctly without this line? Yes → delete it.
> **Anti-pattern:** Don't explain standard concepts (HTTP, JSON, Git). That's padding, not instruction.
> Also: skills are process knowledge, not software — write concrete steps, not DRY abstractions.
>
> Wrong: `def with_retry(fn, max=3): ...` — abstraction, configuration
> Right: `If timeout → wait 5s → retry once → fail? stop.` — one explicit path

### 2. Progressive Disclosure
Three-tier loading: L1 description → L2 SKILL.md body → L3 resources loaded on demand. One reference depth only — every reference linked directly from SKILL.md. No A→B→C reference chains.
**Check:** Pick the deepest fact an agent needs. How many files must it read? >1 → flatten.
> **Anti-pattern:** Deep reference chains force reading 3 files to find one fact. Link everything from SKILL.md.

### 3. Structure Serves Content
Start with just SKILL.md. Add directories only when signaled by ≥3 distinct topics, ≥2 task routes, or a repeated pitfall.
**Check:** Count the directories. Does each have ≥3 non-placeholder files? No → don't add it yet.
> **Anti-pattern:** Scaffolding a full directory tree before content exists (empty references/, hooks/, docs/) wastes context. A directory with 1-2 placeholder files is premature.

### 4. Activation Over Storage
A rule recorded in references/ is dead unless it appears on the agent's execution path — in the routing table, a workflow checklist, or a gotcha. **Check:** Pick a route; trace the read path. Does every critical rule appear?
> **Anti-pattern:** "I already read the SKILL.md earlier" — an agent that skips re-reading misses updated gotchas.

### 5. Defaults Over Menus
For every decision point, provide one default. Mention alternatives only when the default measurably fails. **Check:** Count decision points. Each has a clear default? If "it depends" has no tiebreaker, provide one.

### 6. Procedures Over Declarations
Teach the agent what action to take, not just the desired outcome. "Run validator → fix first error → repeat" is a procedure. "Output must be valid" is a declaration.
**Check:** Find a declaration in this skill. Can you rewrite it as a procedure? Yes → do it. No → you're done.

---

## Quality Checklist

**Before writing, write the eval.** A skill you can't test has no measurable value. Write a concrete task prompt first, then build the skill to pass it.

### P0 — Must pass (skill is broken without these)
- [ ] Task is high-frequency, error-prone, has fixed resources, verifiable criteria
- [ ] Content from real execution traces, not imagination
- [ ] Description has: what + when + user keywords + anti-triggers
- [ ] ≥3 should-trigger and ≥3 should-not-trigger test prompts exist
- [ ] Routing keeps only critical branches; each verifiable by tool/schema/user

### P1 — Should pass (skill degrades without these)
- [ ] SKILL.md ≤ 500 lines
- [ ] Progressive disclosure: core in body, long content in references/
- [ ] One reference depth (all linked from SKILL.md)
- [ ] Repeated error-prone steps are scripted, not inline
- [ ] Safety stop conditions: flag `rm -rf`, `git push --force`, `DROP TABLE` before running scripts
- [ ] Output format specified with concrete template
- [ ] Structural check: frontmatter, naming, layout valid

### P2 — Verification (proves the skill works)
- [ ] Description isolation test: triggers on 3 correct prompts, silent on 3 near-misses
- [ ] Logic validation: every decision point has clear criteria, no ambiguous judgment calls
- [ ] With/without comparison: fewer tool calls, fewer corrections, same/better output
- [ ] Real prompt test: 3-5 diverse prompts through the skill
- [ ] Multi-model test: same prompts on at least two models. Different models interpret triggers differently — a skill that works on one may over-trigger or under-trigger on another.
- [ ] Edge case probes: empty input, missing tool, near-miss triggers, max size
- [ ] One iteration round: fix at least one thing the tests exposed

---

## When to Create a Skill

Create when all 4 hold: (a) repeats across sessions, (b) known error patterns, (c) needs fixed resources, (d) verifiable criteria. Description character budget and installed skill count are coupled constraints — each skill consumes budget at discovery time. Keep descriptions tight and installs to real workflows.

Rule of thumb: the third time you write the same long prompt, consider a skill.

### Decision Tree: Skill vs Prompt vs Reference

```
Repeats across sessions?
├── No → One-time prompt. Stop.
└── Yes → Has error patterns or fixed resources?
    ├── No → Save as a reference snippet (not a skill)
    └── Yes → Has verifiable criteria?
        ├── No → Save as a reusable prompt template
        └── Yes → Create a skill.
```

**Self-generated skills without human review provide no benefit.** The model knows what it wrote. AI handles boilerplate; only human review verifies that gotchas and trigger phrases match reality.

---

## Cross-Harness Thin Shells

Some harnesses lose natural-language instructions after `/compact`. Use a structured routing table in a thin shell file:

```markdown
# AGENTS.md
## Quick Routing
| Task | Skill | Reads |
|---|---|---|
| PDF | pdf | SKILL.md → refs/extraction.md |
```

Place at: `AGENTS.md` (OpenCode/Codex), `CLAUDE.md` (Claude Code), `GEMINI.md` (Gemini), `.cursor/rules/workflow.mdc` (Cursor).

**Critical:** Cursor's `.cursor/skills/<name>/SKILL.md` must have identical description — mismatches cause random activation.

Skill location priority (Codex): `REPO (.agents/skills)` → `USER (~/.agents/skills)` → `ADMIN (/etc/codex/skills)`. Same-name skills at higher priority shadow lower ones silently — check scope when a skill seems ignored.

**Cross-platform:** Same description may trigger differently across harnesses (Codex vs Claude Code vs OpenCode). Test on your target platform.

---

## Anti-Drift Mechanisms

After every non-trivial skill task (30s scan):

- New pitfall → draft gotcha (don't commit until 2nd occurrence)
- Rule ambiguous → flag for update
- Rule outdated → `<!-- DEPRECATED: reason, date -->` + replacement
- Pattern for scripting → note for scripts/ extraction

**Rationalizations table:** Track real excuses the agent has used to skip rules. Source: actual session transcripts, never fabricated.

| Rationalization | Date | Consequence |
|---|---|---|
| "I already read SKILL.md earlier" | 2025-06-12 | Wrong routing, missed gotcha |

**Drift check:** Run the same task on two different project types. `diff` outputs — skeleton should match, business content must differ.

---

## Auditing Third-Party Skills

Before installing an external skill:
1. **Scope**: Does description match your workflows? Never `*` install.
2. **Safety**: Check for credentials, private paths, destructive ops, external callbacks.
3. **Correctness**: Test in an isolated directory first.
4. **Decision**: Install ↔ Modify (narrow triggers, replace rules) ↔ Reject.
> GitHub stars are discovery signals, not safety guarantees.

---

## References

- [Anthropic Agent Skills](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills)
- [agentskills.io specification](https://agentskills.io)
- [OpenAI Codex Skills](https://developers.openai.com/codex/skills)
- [COMPASS Skill Writing Tutorial](https://dongshuyan.com/compass-skills/skill-writing-tutorial.html)
- [Skill Writing Guide by woji_666](https://linux.do/t/topic/1923706)
