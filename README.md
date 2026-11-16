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

## Status

This repository is being built incrementally. Skills should favor clear instructions, evidence-based decisions, and conservative behavior over cleverness or automation for its own sake.

## License

No license has been selected yet.
