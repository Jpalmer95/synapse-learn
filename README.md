# synapse-learn

A skill for agents (Hermes, Claude, Cursor — anything that can read SKILL.md and call an MCP server)
so they can teach a learner through [SynapseJourney](https://synapsejourney.org) and, optionally,
author courses with their own model compute.

- **Teaching**: `list_courses` → `get_course` → teach unit by unit with the unit's own quiz.
- **Authoring**: write a course payload, pass the quality gate, upload. No platform AI is spent.
- **Commons**: published courses are shared — nobody has to regenerate the same course again.

## Install

```bash
git clone https://github.com/Jpalmer95/synapse-learn.git ~/.hermes/skills/synapse-learn
```

Then create a token at https://synapsejourney.org/settings and:

```bash
echo 'SYNAPSE_PAT=sj_your_token' >> ~/.hermes/.env
```

## Contents

- `SKILL.md` — the loadable skill (quickstart, teaching loop, authoring loop, archetypes)
- `references/mcp-tools.md` — every MCP tool with request/response shapes
- `references/course-contract.md` — ingest payload, section budgets, quality-gate rules

## License

Apache-2.0 (skill text). Course content on the platform is CC BY-SA 4.0.
