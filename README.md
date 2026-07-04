# skills

Personal Agent Skills by [pokatomnik](https://github.com/pokatomnik).

This repository contains a curated set of Agent Skills for Zed and other supported agents via the `skills` CLI.

Repository: https://github.com/pokatomnik/skills

## Install

### Install all skills from this repository

```bash
npx skills add github.com/pokatomnik/skills -a zed -y
```

You can also use the full GitHub URL:

```bash
npx skills add https://github.com/pokatomnik/skills -a zed -y
```

### Install one specific skill

```bash
npx skills add github.com/pokatomnik/skills --skill github-issues -a zed -y
```

### List available skills in this repository

```bash
npx skills add github.com/pokatomnik/skills --list
```

### Install locally while developing

From the repository root:

```bash
npx skills add . -a zed -y
```

## Available skills

- `branch-name` — generate a concise branch name from a task or diff
- `caveman` — ultra-compressed communication mode
- `github-issues` — fetch and rank GitHub issues from your own repositories
- `mock-gen` — create a mock from an OpenAPI schema
- `placeholders` — resolve `{{ ... | input }}` placeholders before execution
- `rust-build-optimize` — add Rust build optimization settings to `Cargo.toml`
- `rust-struct-convert` — convert JSON examples into Rust structs
- `shell` — turn natural language into safe shell commands

## Repository structure

The `skills` CLI discovers skills in supported agent directories. For Zed, the layout is:

```text
.agents/skills/
  branch-name/
    SKILL.md
  caveman/
    SKILL.md
  github-issues/
    SKILL.md
  mock-gen/
    SKILL.md
  placeholders/
    SKILL.md
  rust-build-optimize/
    SKILL.md
  rust-struct-convert/
    SKILL.md
  shell/
    SKILL.md
```

Each skill must be a directory containing a `SKILL.md` file with valid YAML frontmatter:

```md
---
name: skill-name
description: Short description of what the skill does and when to use it.
---
```

## Notes

- This repository is public, so anyone can install the skills directly from GitHub.
- The recommended command for this CLI is `npx skills add`.
- If you want to keep skill versions stable, create Git tags or releases.

## License

MIT
