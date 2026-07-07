[English](README.md) | 中文

# Skill Authoring

一个教 Agent 写好 Skill 的 Skill，并且它自己就是自身规则的最佳实践。内容取材于 Anthropic 和 OpenAI 官方文档、agentskills.io 规范，以及真实的社区经验。

## 解决什么问题

如果让 Agent 来写一个 Skill，它通常会产生一大段看起来很专业的文本，可真正用起来就会出问题：
* description 总是触发不了；
* 正文里塞满了显而易见的无用解释，白白占用了上下文；
* Skill 的目录结构生硬地照搬着软件项目的那一套。

本 Skill 所做的，是把"能用"与"好用"的规则区分开，并且拿自身的设计作为示范：以 Task Routing 表格开篇（Activation Over Storage）；正文直截了当地说明了它在教一种结构，同时也展示了另一种（Progressive Disclosure）；每条设计原则都配上了反模式检查，把原则、检验句以及常见失败点放在了同一个阅读区块里。

## 安装

把 `SKILL.md` 复制到你的 Agent 可以找到的 skills 目录下：

| Agent            | 安装路径                          |
| ---------------- | --------------------------------- |
| OpenCode / Codex | `~/.agents/skills/skill-authoring/` |
| Claude Code      | `~/.claude/skills/skill-authoring/` |
| Cursor           | `.cursor/skills/skill-authoring/`   |

或者直接 clone 整个仓库：

```bash
git clone https://github.com/Ronifue/skill-authoring.git ~/.agents/skills/skill-authoring
```

## 内容

单一文件，一共 328 行。以 Task Routing 表格开篇，可以直接跳转到需要的章节。Quickstart 提供了一份可直接复制的极简 skill 骨架。

六条设计原则，每条都配了 Check 检验句和内联的反模式警告，原则、检验和常见失败点放在同一个阅读区块里。还附带了一个决策树，帮你判断是该写 skill、保存为参考片段，还是采用一次性 prompt。Degrees of Freedom 框架则教你依据任务脆弱程度来选取正确的 skill 形态。

四种常见模式，都附上了具体实例（坑点索引、模板、检查清单、验证循环）。一份 P0-P2 质量检查清单，可以用来做自检。还涵盖了跨 harness 薄壳、反漂移维护、第三方 skill 审计等方面的内容。

## 与其他提示词编写类 Skill 的关系

其他用于编写提示词的 Skill 负责的是语言层面：怎么措辞、怎么编写 XML 段落、怎么处理安全边界。本 Skill 负责的则是架构层面：文件放在哪里、路由怎么设计、渐进披露怎么分层。两者是正交的——可以同时使用，不过不同任务应当触发不同的 Skill。

## 参考来源

本 Skill 汇集了以下来源的知识：

- [Anthropic Agent Skills 文档](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills)
- [OpenAI Codex Skills 指南](https://developers.openai.com/codex/skills)
- [agentskills.io 规范](https://agentskills.io)
- [COMPASS Skill 编写教程](https://dongshuyan.com/compass-skills/skill-writing-tutorial.html) by dongshuyan
- [Linux DO - 【这里是沃基】如何写一个好的skill 让你的效率加倍](https://linux.do/t/topic/1923706) by woji_666
- agentpatterns.ai、Perplexity 研究、mgechev 最佳实践等社区模式

更多内容可以在 [Linux DO](https://linux.do/) 上面探索。

License: MIT
