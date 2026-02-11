# Agent Skills

Skills are pre-built instruction sets for performing complex, multi-step Tuist tasks. Instead of manually guiding an agent through a migration or setup process, you install a skill and let the agent handle it.

## Available skills

### Migrate to Tuist

Migrates existing Xcode projects to Tuist-generated workspaces with build and run validation, external dependency mapping, and migration checklists.

```bash
SKILL_NAME=tuist-migrate
SKILL_URL=https://tuist.dev/skills/migrate/SKILL.md
```

## Installation

Each coding agent has its own mechanism for loading skills. Below you will find instructions for the most popular ones. The examples use `$SKILL_NAME` and `$SKILL_URL` variables listed alongside each skill above.

### Claude Code

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) loads skills from `SKILL.md` files inside a `skills/` directory.

**Project:**
```bash
mkdir -p .claude/skills/$SKILL_NAME
curl -o .claude/skills/$SKILL_NAME/SKILL.md $SKILL_URL
```

**Global:**
```bash
mkdir -p ~/.claude/skills/$SKILL_NAME
curl -o ~/.claude/skills/$SKILL_NAME/SKILL.md $SKILL_URL
```

Once installed, invoke it with `/$SKILL_NAME` inside a Claude Code session.

### Codex

[Codex](https://github.com/openai/codex) loads skills from `SKILL.md` files inside a `skills/` directory. It searches the repository root, current working directory, and parent folders.

**Project:**
```bash
mkdir -p .codex/skills/$SKILL_NAME
curl -o .codex/skills/$SKILL_NAME/SKILL.md $SKILL_URL
```

**Global:**
```bash
mkdir -p ~/.codex/skills/$SKILL_NAME
curl -o ~/.codex/skills/$SKILL_NAME/SKILL.md $SKILL_URL
```

### Amp

[Amp](https://ampcode.com) supports skills through `SKILL.md` files in a `skills/` directory, and also reads `AGENTS.md` for general instructions.

**Project:**
```bash
mkdir -p .agents/skills/$SKILL_NAME
curl -o .agents/skills/$SKILL_NAME/SKILL.md $SKILL_URL
```

**Global:**
```bash
mkdir -p ~/.config/agents/skills/$SKILL_NAME
curl -o ~/.config/agents/skills/$SKILL_NAME/SKILL.md $SKILL_URL
```

### OpenCode

[OpenCode](https://opencode.ai) loads skills from `SKILL.md` files inside a `skills/` directory.

**Project:**
```bash
mkdir -p .opencode/skills/$SKILL_NAME
curl -o .opencode/skills/$SKILL_NAME/SKILL.md $SKILL_URL
```

**Global:**
```bash
mkdir -p ~/.config/opencode/skills/$SKILL_NAME
curl -o ~/.config/opencode/skills/$SKILL_NAME/SKILL.md $SKILL_URL
```

---

This repository is automatically updated from [tuist/tuist](https://github.com/tuist/tuist).
