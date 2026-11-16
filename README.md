# Agent Skills

A public collection of reusable [Agent Skills](https://agentskills.io/) for coding agents.

The collection is intended for workflows that need more context than a prompt template but do not require a separate executable integration. Skills can be used with Pi and other Agent Skills-compatible tools.

## Repository layout

```text
agent-skills/
└── skills/
    └── <skill-name>/
        ├── SKILL.md
        ├── references/   # optional
        ├── scripts/      # optional
        └── assets/       # optional
```

Each skill must have a `SKILL.md` with valid frontmatter containing at least `name` and `description`.

## Available skills

- [`vibetrap`](skills/vibetrap/SKILL.md) — a slow, conservative review workflow for understanding architecture and business logic before approving AI-assisted code changes.

## Using a skill with Pi

Clone the repository, then copy or link the skill directories into a Pi-discoverable skills directory such as `~/.agents/skills/` or a project's `.agents/skills/` directory.

```bash
git clone https://github.com/jerichosiahaya/agent-skills.git
cp -R agent-skills/skills/<skill-name> ~/.agents/skills/
```

Use `/skill:<skill-name>` to invoke a skill explicitly when supported by the host.

## Agent installation prompt

Copy and paste this prompt into your coding agent to install `vibetrap`:

```text
Install and activate the `vibetrap` Agent Skill from https://github.com/jerichosiahaya/agent-skills.

First determine which skill directory your runtime uses. Prefer the project-local skill directory when I want this enabled only for the current repository; otherwise use the user-level directory. For Pi, use `.agents/skills/` in the project or `~/.agents/skills/` globally.

Install only `skills/vibetrap/` as `<skill-directory>/vibetrap/`, preserving the existing `SKILL.md` if one already exists. Do not overwrite an existing installation without showing me the difference and asking first. Do not modify application source code, project configuration, or unrelated files.

After installation:
1. Read `vibetrap/SKILL.md` completely.
2. Verify that its frontmatter contains `name: vibetrap` and a description.
3. Confirm the exact installed path and the repository commit or version used.
4. Explain how to invoke it in this runtime, such as `/skill:vibetrap`.

If this runtime does not support Agent Skills, do not pretend the installation worked. Tell me what is unsupported and provide the manual installation path instead.
```

## Status

This repository is being built incrementally. Skills should favor clear instructions, evidence-based decisions, and conservative behavior over cleverness or automation for its own sake.

## License

No license has been selected yet.
