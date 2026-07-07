English | [中文](README.zh.md)

# Skill Authoring

A skill that teaches agents how to write better skills — and is itself the best example of its own rules. Drawn from Anthropic and OpenAI official documentation, the agentskills.io specification, and real-world community experience.

## What this is

When you ask an agent to create a skill, it often produces a wall of text that looks professional but breaks down in practice:

* descriptions that never trigger;
* bodies cluttered with obvious explanations, burning through context;
* directory structures blindly copied from software projects.

This skill does one thing: it tells usable apart from good. And it uses its own design as the example — a Task Routing table up front (Activation Over Storage), a body that bluntly explains it's teaching one structure while also showing another (Progressive Disclosure), and every design principle paired with an anti-pattern check that puts the rule, the test, and the failure modes in one reading block.

## Installation

Copy `SKILL.md` into your agent's skills directory:

| Agent | Install path |
|---|---|
| OpenCode / Codex | `~/.agents/skills/skill-authoring/` |
| Claude Code | `~/.claude/skills/skill-authoring/` |
| Cursor | `.cursor/skills/skill-authoring/` |

Or clone the repository:

```bash
git clone https://github.com/Ronifue/skill-authoring.git ~/.agents/skills/skill-authoring
```

## What is inside

A single-file skill, 328 lines in total. Opens with a Task Routing table so you can jump straight to what you need. A Quickstart gives you a minimal copy-paste skill skeleton.

Six design principles, each paired with a Check question and an inline anti-pattern warning. The rule, its test, and the common failure modes sit together in one reading block. A Decision Tree maps whether to create a skill, save a reference, or use a one-time prompt. A Degrees of Freedom framework teaches you to pick the right skill shape based on task fragility.

Four common patterns, each with concrete examples (gotchas, templates, checklists, validation loops). A P0-P2 quality checklist you can use for self-testing. Also covers cross-harness thin shells, anti-drift maintenance, and auditing third-party skills.

## Relationship to prompt-writing skills

Skills that handle prompt authoring take care of language: phrasing, XML sections, safety boundaries. This skill takes care of architecture: file placement, routing design, progressive disclosure. The two are orthogonal — use both, just trigger them for different tasks.

## Credits

This skill draws from:

- [Anthropic Agent Skills documentation](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills)
- [OpenAI Codex Skills guide](https://developers.openai.com/codex/skills)
- [agentskills.io specification](https://agentskills.io)
- [COMPASS Skill Writing Tutorial](https://dongshuyan.com/compass-skills/skill-writing-tutorial.html) by dongshuyan
- [Linux DO — How to Write a Good Skill](https://linux.do/t/topic/1923706) by woji_666
- Community patterns from agentpatterns.ai, Perplexity research, and mgechev's best practices

Learn more on [Linux DO](https://linux.do/)

License: MIT
