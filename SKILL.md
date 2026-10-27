---
name: synapse-learn
description: Use when a user wants to learn a subject through SynapseJourney, or wants an agent to author and upload courses into it.
version: 1.0.0
author: Jonathan Korstad
license: Apache-2.0
---

# SynapseJourney — learn with your agent

SynapseJourney (https://synapsejourney.org) is a learning platform where the **agent does the
teaching** and the platform stores the curriculum, the progress, and the 3D knowledge map. There is
no platform AI in the loop: your model explains and grades, the site renders and tracks.

Use this skill when the user says things like:

- "teach me orbital mechanics" / "get me up to speed on X this week"
- "what courses are there?" / "continue my course on Y"
- "write me a course on Z" / "add a unit about W to that course"
- anything about the SOTA bottleneck registry or the Open Science hypotheses

## Quickstart (once per machine)

1. User creates a token: https://synapsejourney.org/settings → **Create Hermes token** (`sj_…`, shown once).
2. Store it and point at the deployment:

```bash
echo 'SYNAPSE_PAT=sj_your_token_here' >> ~/.hermes/.env
echo 'SYNAPSE_BASE_URL=https://synapsejourney.org' >> ~/.hermes/.env
```

3. Verify:

```bash
curl -sS -H "Authorization: Bearer $SYNAPSE_PAT" "$SYNAPSE_BASE_URL/api/auth/user"
```

Never echo the token back to the user or write it into a repo.

## Two ways to talk to it

**MCP (preferred).** Streamable-HTTP JSON-RPC at `$SYNAPSE_BASE_URL/api/mcp`, bearer PAT.

```json
{
  "mcpServers": {
    "synapse": {
      "type": "http",
      "url": "https://synapsejourney.org/api/mcp",
      "headers": { "Authorization": "Bearer sj_your_token" }
    }
  }
}
```

Tools: `synapse.list_courses`, `synapse.search_knowledge`, `synapse.get_course`,
`synapse.ingest_course`, `synapse.list_sota_bottlenecks`, `synapse.propose_sota_bottleneck`,
`synapse.submit_hypothesis`, `synapse.get_3d_map`. See `references/mcp-tools.md`.

**REST (when there is no MCP client).** Same auth; `/api/learn/my-courses`, `/api/topics`,
`/api/lessons/unit/<id>/content`, `/api/learn/ingest`, `/api/open-science/bottlenecks`.

Machine-readable discovery: `GET /.well-known/synapse.json` (endpoints, archetypes, auth, corpus size).

## The teaching loop

```
1. synapse.list_courses                     → pick by title/category, note topicId
2. synapse.get_course {topicId}             → coursePlan.archetype + units in teaching order
3. For each unit:
     GET /api/lessons/unit/<unitId>/content → concept, example, quiz, resources
     Teach it in your own words. Do not dump the markdown verbatim.
     Ask the unit's quiz questions before moving on; explain wrong answers.
4. Finish: synapse.submit_hypothesis        → log what the learner found interesting/unsolved
```

Rules that make this work rather than annoy:

- **One unit at a time.** The platform's units are already sized (~1 sitting each); dumping eight
  units is a wall of text, not teaching.
- **Use the unit's own quiz.** It matches the prose; inventing new questions drifts off-curriculum.
- **Say where you are.** "Unit 3 of 8, ~20 minutes" gives the learner a stop point.
- **Respect the depth mode.** `contentView: "skim"` means they chose speed over depth.

## The authoring loop (BYOC)

Author with your own compute, then upload — the server never calls a model.

```bash
curl -sS -X POST "$SYNAPSE_BASE_URL/api/learn/ingest" \
  -H "Authorization: Bearer $SYNAPSE_PAT" -H 'Content-Type: application/json' \
  -d @course.json
```

Payload shape, archetype budgets, and the quality rules are in `references/course-contract.md`.
Two behaviours worth knowing before you try:

1. **Quality gate (422).** Units with template prose, repeated quiz stems, placeholder/search-page
   resource URLs, or prose far outside the archetype band are rejected with a per-unit report. Fix
   the flagged units; do not resend the same payload.
2. **Duplicate gate (409).** If a *published* course already covers that subject, extend it instead:
   resend with `topicId` set to append a modular add-on. New topics land **private to the author**
   and stay out of the public catalogue until reviewed.

## Course archetypes (pick the one that matches the ask)

| archetype | units | words/section | use |
|---|---|---|---|
| `rapid_bootstrap` | 3–5 | 140–240 | unblock a task today |
| `focused_skill` | 6–10 | 200–320 | one capability this week |
| `foundation` | 18–28 | 420–650 | real underlayment in a science/EE field |
| `deep_dive` | 12–18 | 350–520 | one niche, properly |
| `frontier_survey` | 6–10 | 260–380 | where a field is stuck now |
| `cross_domain` | 8–14 | 320–480 | two fields plus a capstone |

"Make it quick" → `rapid_bootstrap`. "I want to actually understand physics" → `foundation`.
Getting this wrong is the most common authoring mistake: a foundation course with 200-word
sections is as broken as a rapid bootstrap with 700-word essays.

## Never

- Never send model API keys, OAuth tokens, or session cookies to the platform — only the PAT.
- Never upload a course the user did not ask for, or publish on their behalf.
- Never claim progress was recorded without checking (`get_course` shows completed units).
