# Class 09 — CI Workers, Plugin Authoring & Project

**Source:** Panaversity, *09. CI Workers, Plugins Authoring & Project* (YouTube, unlisted, 1:52:20, uploaded Oct 25, 2026)
**Series:** FDE 1 & 2 classes, Claude Architect exam prep
**Language:** Urdu/Hindi. These notes are a summary in English.

---

## Overview

| Part | Timestamp | Topic |
|---|---|---|
| 1 | 0:00 – 52:10 | Revision of CI workers, triggers, and a scheduled (cron) Claude worker |
| 2 | 52:10 – 1:21:09 | Plugins: marketplace, installing, authoring, shipping |
| 3 | 1:21:09 – 1:40:58 | Project 2: Email Marketing Studio |
| 4 | 1:40:58 – end | Q&A |

---

## 1. Recap of the previous class (0:00)

- Last class covered **Claude on the web**, then **Claude Code**. Claude Code is about **20% of the exam weight**.
- Topics already covered: Claude Code basics, setting up a project *as an architect* (skills, MCP connectors, rules, telling Claude about the project).
- The new idea: Claude doesn't have to run only on your own machine. The most common place to run it elsewhere in software is **CI/CD**.

### What is CI (Continuous Integration)? (1:34)
- An **event** (commit, push, PR) triggers an automated **script/workflow**.
- Example from the last class: a **Markdown bot** ran on every PR, checked that only `.md` files were changed, marked files ✅/❌, and **blocked the merge** if any check failed.
- **Analogy:** CI is a *house rule*. When you add a new ingredient to the soup, you taste it before serving. **GitHub Actions** is the *helper* that does the tasting.
- GitHub Actions is just one option; other CI runners exist.

### The key insight: the robot has a VM (7:37)
- Every GitHub Actions "robot" runs on a **virtual machine**.
- A VM can run anything your physical machine can, including **Claude Code, OpenCode, or a custom agent**.
- So Claude Code can run in three places:
  1. Your local machine
  2. Anthropic's cloud (claude.ai, the **Code** tab)
  3. A CI virtual machine (GitHub Actions)
- The project lives on GitHub, where developers do more than write features: they **review code, assign tasks, discuss architecture**. Claude can do all of these on GitHub too.

### Setting it up (11:14)
- Claude Code has a built-in command, **`/install-github-app`**, that sets up the workflows automatically.
- With OpenCode or other agents, the setup is more manual, but the concept is the same. Tip: use the skills marketplace together with the GitHub Actions docs.
- Workflows go in **`.github/workflows/*.yml`**.

### The 4 minimum parts of every workflow file (13:43)
1. **`name`** — what the workflow is called
2. **Trigger (`on:`)** — when it runs
3. **Machine (`runs-on:`)** — e.g. `ubuntu-latest`
4. **`steps`** — what it does (checkout, run Claude with a prompt, etc.)

Plus **permissions**, and an optional **prompt/instructions** for Claude.

### The three robots in the demo repo (21:10)

| # | Robot | Trigger | Job |
|---|---|---|---|
| 1 | **Markdown check** | PR | Checks for empty or non-markdown files (scripted, no AI) |
| 2 | **Claude (interactive)** | Issue/comment that tags `@claude` | Reads the user's input, completes the task, opens a PR, reports back |
| 3 | **Claude Code Review** | PR opened | Reviews all changes like a human reviewer; pass/fail |

- PR flow: the **auto check** (markdown) runs, then the **live check** (Claude review). One check can be *required* and the other *optional*.
- In the Actions log you can see: VM setup, then repo checkout, then Claude install (via Bun), then Claude runs with the generated prompt. **As an architect, read the prompt** to see what Claude is actually told.

### Q: Does each robot get its own machine? (23:58)
- **Yes.** Each run gets a **fresh, separate VM**. It spins up, does the work, and is **destroyed** at the end. VMs are not shared or reused.

---

## 2. What can trigger Claude? (23:45)

| # | Trigger type | Example |
|---|---|---|
| 1 | **GitHub events** | push, PR created |
| 2 | **Tagging `@claude`** | comment on an issue or PR |
| 3 | **Schedule (cron)** | every day at 9 PM |
| 4 | **Manual run** (`workflow_dispatch`) | click "Run workflow" to test |
| 5 | **External system** | an email worker or another app triggers it (covered later, when building MCP servers) |

> The *task* doesn't matter (send an email, write notes, fix an issue). What matters is **when** it runs.

### Demo: a scheduled "Daily Lesson" worker (27:24)
- Prompt to Claude: *Can we set up a GitHub Action to run Claude Code on its own schedule for a particular task? Example: add a lesson file daily at 9 PM.* Ask it to **explain first**, then go into details.
- Claude's plan:
  - A new **4th workflow file** in `.github/workflows/`
  - Trigger: **`schedule:` with a cron expression** (minute, hour, day of month, month, day of week)
  - ⚠️ **Time zone:** cron runs in UTC, so convert from your local time (PKT, UAE, US, etc.)
  - Machine: `ubuntu-latest`
  - Claude setup: auth was already configured by `/install-github-app`
- Claude asked clarifying questions:
  - Push straight to `main`, or open a PR? → *main*
  - Who picks the topic? → *Claude decides*
  - Time zone? → *Pakistan, 9 PM*
  - What if yesterday's lesson didn't merge? → *doesn't matter, work on main*
- **Issue:** `main` was **branch-protected** (no direct pushes). Fix: **PR + auto-merge**.

### Key `claude_args` in the workflow (37:17)
- **`--max-turns 40`**: the maximum number of agent-loop steps (actions such as web search, create file, write file). This is a **spending/attempt limit**. If the work isn't done in 40 turns, it stops.
- **`--allowed-tools`**: which of Claude's 30–40+ tools it may use.
- **`prompt`**: the full instructions. This is **the most important part**, because Claude works from it.

### Result (39:42)
- Merged the PR; the **Daily Lesson** action now appears in the Actions tab.
- Ran it manually to test. Claude created a lesson ("Tool use and agentic loops"), opened a PR, and it was merged.
- Lesson writing can take 5–10+ minutes.

### Where does the model come from? (47:27)
- Claude Code **can't run without a model**. It needs a login or an API key.
- `/install-github-app` connected Claude to GitHub using the **existing subscription**, so no separate setup was needed. It uses the **default model** (Sonnet in this case).
- In the workflow file you can use **any model or LLM**: a different provider, a raw LLM API call, an open-source harness. **It's just a machine; whatever you write in the file runs there.**

### Big picture (43:20)
- This is the same idea as **scheduled tasks on claude.ai**, moved to your own infrastructure (GitHub today; any cloud, VM, or container tomorrow).
- With a well-built **plugin + skills + workflow**, you can take a prebuilt harness and **ship a worker anywhere**.
- This is **tested in the exam** and is real architect work: automated code reviews, automated feature requests, scheduled agents.
- 📌 **Practice it.** The repo and prompts are shared in the class group.

---

## 3. Plugins (52:10)

### What is a plugin?
- A **package of reusable domain expertise**. It can contain **any or all** of:
  1. **Agent skills**
  2. **Connectors** (MCP servers / tools)
  3. **Hooks**
  - Plus extras such as slash commands, status lines, etc.
- Plugins work with **Claude Code and on the web**.

### Plugin marketplace in Claude Code (53:43)
- Run **`/plugins`** to manage plugins:
  - **Discover**: browse what's available
  - **Installed**: what you have and its status
  - **Marketplaces**: the default is **Anthropic's directory**; you can add others (custom, prebuilt)
- Install scope: **user**, **project collaborators**, or **this project only**.
- Option to **review**, which opens the plugin's repo so you can inspect or scan it first.
- Demo: installed **"brief-interview"**, a single `SKILL.md` that interviews a client before writing a project brief. After install, it shows up as a skill.
- 📝 **Homework:** run Claude → `/plugins` → Marketplaces → Anthropic directory → explore.

### Skills vs. Plugins: side by side (1:00:53)

| | Skills | Plugins |
|---|---|---|
| Contains | A procedure | Skills + connectors + hooks |
| Marketplace | skills.sh (third party) | `/plugins` marketplaces |
| Builder skill | **skill-creator** | **plugin-authoring** |

### How to build a plugin (1:02:04)
1. Decide what the plugin is **for**.
2. Build each piece **separately first**: skills (via skill-creator), MCPs, hooks. **Test them** and make sure they work in a real project.
3. **Package** them into a plugin.

> **Packaging is the easy part.** The real effort is in the components: does the skill actually work, and is the connector reliable? Evals and testing start with the individual skills.

### Q: Zia Tutor AI has a skill + MCP in a zip. Why isn't it a plugin? (1:05:32)
- Because it was **never packaged or bundled** as one. It's individual components set up like a normal project.
- It wasn't needed, since most Tutor users are on the **web**.

### Demo: packaging Zia Tutor as a plugin (1:07:38)
- Used the **plugin-authoring** skill. It can build standalone plugins and also **customize Claude Code itself** with small plugins.
- Claude jumped straight to building. Tip: use **Plan mode** (`Shift+Tab`) and say *"discussion only session, ask questions using AskUserQuestion"*.
- Output: only one new file, **`plugin.json`** (name, version, description), bundling the existing skill + MCP config.
- **Shipping it:** "I want it as a GitHub repo so everyone can install and use it." Claude used **`gh` CLI** to create the repo and gave an **install command** (`/plugin install <name>@<marketplace>`).
  - The more **agent-native** your tools are (git, gh CLI), the better agents can use them.
- After install, the skill appears as `pluginName:skillName` and works the same way.

### 📝 Challenge
> **Ship Zia Tutor AI as a plugin yourself**, using the plugin-authoring skill.

---

## 4. Project 2: Email Marketing Studio (1:21:09)

### Goal
Build a small **email-marketing studio** run by **Claude Code** (or OpenCode). Claude will:
- Plan the **campaign** and how to launch it
- **Write emails**, personalized per target segment
- Optionally **send** them via a practice email server

Example idea: you know how to manage an AI worker (from the first classes) and want to reach people who'd pay for it.

### Suggested folder structure
```
email-marketing-studio/
├── CLAUDE.md          # project context
├── brand-voice.md     # (or move into .claude/rules/)
├── leads/             # who to email (+ its own CLAUDE.md)
├── campaigns/         # saved progress/output
├── .claude/           # hooks, skills, rules
├── .mcp.json          # MCP config
└── scripts/           # any scripts Claude writes
```
- Reminder: there can be **multiple `CLAUDE.md` files**. Each one loads when Claude works in that folder.

### Prerequisites
Node, Git, **GitHub CLI (`gh`)**, Claude Code.

### Workflow for every step
1. Understand the step
2. Give Claude the provided **prompt**
3. Run the provided **check prompt** to verify the result matches the requirement
4. Ask questions, then move to the next step

### Steps demoed (1:28:33)
- **Step 0:** create the folder → init Claude → *"Set up GitHub repo and push this project there"* (public). Good habit for every project.
- **Step 1 (research only):** have Claude check candidate skills on **skills.sh**, recommend which to use or skip and why, and write notes. **Don't install anything.**
  - Claude recommended Anthropic's official campaign skill, Resend's official email-sending skill, and a marketing skill.
  - Tip: **`!` prefix** runs a shell command from inside Claude (e.g. `!code .` to open VS Code).
- **Step 2 (setup):** install Anthropic's **marketing plugin**, create the structure (brand, leads, campaigns, notes), write `CLAUDE.md` + `.gitignore`, push to GitHub, then verify with the check prompt.
- Email marketing is just the **running example**. The real goal is learning the **configuration**.

### Project parts
| Part | Content |
|---|---|
| 1 | Basic folder setup + skills |
| 2 | Live services + **safety & security** |
| 3 | Automation, workflows, automatic reviews, **package the studio kit as a plugin** |
| Extra | Optional challenges |

- **Required:** through Part 3. Includes a **mapping to exam objectives**.
- ⏰ **Deadline: ~2 weeks** (by the Sunday after next). Docs and prompts will be on the dashboard.
- Get unstuck with your coding agent first, then the class group.

### Exam notes (1:40:10)
- Everything covered so far is exam material: **Agentic coding, Claude Code for teams, Claude Code as CI worker**.
- **Routines** aren't covered yet. The official updated guide is expected **Nov 17**; new topics will be added then.

---

## 5. Q&A (1:41:33)

**Q: If I build in the Claude Desktop app, are files local or in a VM?**
- With a **local project** in the desktop app's Code tab, files are **on your machine**. Desktop app or CLI doesn't matter. Class uses the CLI because many students use OpenCode.

**Q: If I vibe-code an ERP, what hurdles come with scale?**
- Vibe coding is fine for **demos and small tasks**, but not for **production**.
- Agents will handle the mechanical part (files, code), and models keep improving. But **architecture and design decisions are still yours**.
- Without architect skills you can build an MVP but **can't extend it**. Ship one feature and another breaks, and you lose reputation.
- Shipping production apps *with* coding agents is a skill in itself. That gap is why **FDE roles are in demand**.

**Q: Zia Tutor installed via connector on Claude: is that a plugin or a connector?**
- **Not a plugin.** It's a connector + a skill added separately. Without the skill it still works (good models compensate), but teaching quality drops slightly.
- Tip: after every new model release, **audit your repos** for what to remove or keep.

**Q: Same instructions work in project A but not project B (desktop app). How do I troubleshoot?**
1. Put **both projects on GitHub** first, so nothing is lost if Claude changes something.
2. Open a Claude session, point it at both, and ask: *what exists in project 1 but not project 2?*
3. Likely causes: **config** or **memory** differences, if the model, effort level and prompt are the same.
4. Ask Claude *"what's saved in this project's memory, and where is the file?"* Then copy the useful parts into the other project's `CLAUDE.md`, or turn them into a **skill**.

---

## ✅ Action items
- [ ] Practice the CI worker setup (review bot, `@claude` bot, scheduled bot)
- [ ] Explore `/plugins` → Anthropic marketplace
- [ ] Ship **Zia Tutor AI as a plugin** with the plugin-authoring skill
- [ ] Build **Project 2: Email Marketing Studio** (Parts 1–3) within ~2 weeks
