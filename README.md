# yunn-skills

English | [中文](./README.zh.md)

A collection of reusable Agent Skills for structured planning and long-form fiction writing. Designed for Codex and other clients compatible with the [Agent Skills specification](https://agentskills.io/specification).

## Prerequisites

- An AI coding agent with Agent Skills support, such as Codex
- Node.js and `npx` for the recommended installation method

## Installation

> **Tip:** Install only the skills you need. Every enabled skill adds metadata to your agent context.

### Quick Install (Recommended)

```bash
npx skills add Yunshiro/yunn-skills
```

Follow the prompts to select `html-plan`, `novel-writer`, or both.

### Ask Codex to Install

Invoke the built-in installer in Codex:

```text
$skill-installer Install html-plan and novel-writer from https://github.com/Yunshiro/yunn-skills
```

### Codex Project-Level Install

To make the skills available only inside one project, copy the complete skill directories into `<project>/.agents/skills`:

```bash
git clone https://github.com/Yunshiro/yunn-skills.git
mkdir -p /path/to/your-project/.agents/skills
cp -R yunn-skills/skills/html-plan /path/to/your-project/.agents/skills/
cp -R yunn-skills/skills/novel-writer /path/to/your-project/.agents/skills/
```

The resulting structure should look like this:

```text
<project>/.agents/skills/html-plan/SKILL.md
<project>/.agents/skills/novel-writer/SKILL.md
```

### Codex User-Level Install

To use the skills across all projects, copy them into `~/.agents/skills`:

```bash
git clone https://github.com/Yunshiro/yunn-skills.git
mkdir -p ~/.agents/skills
cp -R yunn-skills/skills/html-plan ~/.agents/skills/
cp -R yunn-skills/skills/novel-writer ~/.agents/skills/
```

Codex normally detects skill changes automatically. Restart Codex if a newly installed skill does not appear.

## Usage

Codex can activate a skill in two ways:

1. **Explicit invocation:** mention the skill with `$skill-name`.
2. **Automatic activation:** describe a matching task and let Codex select the skill from its description.

For predictable results, explicitly invoke the skill and include the goal, context, scope, length, and expected output in your prompt.

## Available Skills

### html-plan

Create a polished, standalone HTML planning document for technical and project work.

```text
$html-plan Create an implementation plan for migrating a SaaS application
to a multi-tenant architecture. Include scope, milestones, owners,
dependencies, risks, rollback, and acceptance criteria.
```

**Best for:**

- Implementation and technical execution plans
- Project and roadmap plans
- Migration and rollout plans
- Release plans
- Research and investigation plans

**Features:**

- Produces one self-contained `.html` file
- Uses semantic HTML with consistent heading levels
- Adds a table of contents for longer documents
- Uses tables for timelines, owners, dependencies, risks, and status
- Includes a neutral document theme and print-friendly CSS
- Works directly from the filesystem without external fonts, scripts, or build tools

This skill is intended for execution documents. It is not designed for marketing pages, dashboards, or interactive web applications.

### novel-writer

An end-to-end fiction writing workflow covering story discovery, character design, outlining, chapter drafting, revision, and long-form continuity.

```text
$novel-writer Write an 8-chapter, 20,000-word urban fantasy novel for
Fanqie Novel. The protagonist is a night-shift designated driver who can see
the last ownership transfer of any object he touches.
```

Literary fiction example:

```text
$novel-writer Create a 30,000-word realist novella for a literary contest.
Set it in a fading mining town in northern China, use third-person limited,
and keep the prose restrained.
```

**Two writing modes:**

| Mode | Best for | Focus |
| --- | --- | --- |
| Commercial web fiction | Fanqie, Qimao, Qidian, and similar platforms | Strong opening chapters, payoff pacing, escalating conflict, immersion, and anti-AI prose checks |
| Literary fiction | Contests, literary publications, and personal projects | Character arcs, value shifts, subtext, restraint, and open-ended resonance |

**Workflow:**

1. Gather requirements and determine the writing mode.
2. Create character DNA profiles.
3. Build the story outline and conflict structure.
4. Generate chapter-level emotional beats.
5. Draft the novel chapter by chapter or volume by volume.
6. Revise for immersion, AI-like patterns, pacing, dialogue, setting, and interiority.

Projects with 12 chapters or fewer and under 50,000 words are handled chapter by chapter. Projects with 13 or more chapters, or 50,000 words or more, switch to volume-based planning and maintain progress, memory, and foreshadowing records for continuity across sessions.

**Default output:**

```text
output/
├── 章节正文/
│   ├── 第1章·章节标题.md
│   └── ...
└── 创作材料/
    ├── 书名-角色DNA档案.md
    ├── 书名-故事大纲.md
    ├── 书名-情绪节拍表.md
    └── ...
```

The skill pauses at confirmation points so you can review the platform, premise, characters, outline, and beats before drafting continues.

## Repository Structure

```text
yunn-skills/
├── README.md
├── README.zh.md
└── skills/
    ├── html-plan/
    │   └── SKILL.md
    └── novel-writer/
        ├── SKILL.md
        ├── agents/
        │   └── openai.yaml
        ├── references/
        │   ├── commercial-mode.md
        │   ├── literary-mode.md
        │   ├── long-form-continuity.md
        │   ├── planning.md
        │   └── writing-and-revision.md
        └── scripts/
            └── check_novel.py
```

Each skill is self-contained and can be installed independently. `novel-writer` loads detailed references only when the corresponding writing stage needs them and uses one deterministic script for both chapter and manuscript checks.

## Updating

If you installed with `npx skills`, run the installation command again to fetch the latest version:

```bash
npx skills add Yunshiro/yunn-skills
```

For a manual installation, pull the repository and copy the required skill directories again. Check for local customizations before overwriting an existing installation.

## References

- [OpenAI: Build skills](https://learn.chatgpt.com/docs/build-skills)
- [Agent Skills specification](https://agentskills.io/specification)
