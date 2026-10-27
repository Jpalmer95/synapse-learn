# SynapseJourney MCP tools

Endpoint: `POST https://synapsejourney.org/api/mcp`
Transport: Streamable HTTP (protocol `2025-03-26`, also answers `2024-11-05`)
Auth: `Authorization: Bearer sj_<token>`
Session: server returns `Mcp-Session-Id` on `initialize`; echo it back on later calls.
`GET /api/mcp` returns 405 by design (no server-initiated stream). `DELETE /api/mcp` ends a session.

## synapse.list_courses

```json
{ "category": "Physics", "limit": 60 }
```

Returns published courses plus the caller's own private ones:

```json
[{"topicId":16,"title":"Electromagnetism","category":"Physics","units":8,
  "archetype":"focused_skill","qualityScore":96,"published":true,
  "courseUrl":"/rabbit-hole?topic=16"}]
```

Use this first — it is cheaper than search and tells you what is actually teachable.

## synapse.search_knowledge

```json
{ "query": "common-mode EMI wide bandgap", "limit": 8 }
```

Hybrid vector + trigram search over topics, bottlenecks, and OER resources. Falls back to
trigram when the deployment has no embedding backend configured. Only returns topics the caller
may see.

## synapse.get_course

```json
{ "topicId": 16 }
```

Returns `topic`, `coursePlan` (including `planJson.archetype`), `units` **in pedagogical order**
(beginner → nextgen), linked `bottlenecks`, and `resources`. Unit objects include `id`, `title`,
`outline`, `difficulty`, `unitIndex`, and `contentJson` when content exists.

Teaching detail lives at `GET /api/lessons/unit/<unitId>/content` — same rows the site renders.

## synapse.ingest_course

```json
{
  "topicId": 16,
  "topic": { "title": "Electromagnetism" },
  "learningIntent": "standard",
  "units": [{ "title": "...", "difficulty": "advanced", "outline": "...", "contentJson": { } }]
}
```

- Omit `topicId` to create a new course (it lands private to the author).
- Pass `topicId` to append units to an existing course — the modular add-on rail. Existing titles
  are skipped, progress is untouched.
- Fails with `-32602` and a quality report if the gate rejects the units.

## synapse.list_sota_bottlenecks / propose_sota_bottleneck

```json
{ "domain": "Power Electronics", "limit": 20 }
```

`propose` body: `domain, title, description, whyItMatters, currentApproaches[], openQuestions[],
topicId`. Use it when a lesson surfaces a real unsolved limit — that is the point of the registry.

## synapse.submit_hypothesis

```json
{
  "title": "...", "content": "mechanism + rationale",
  "hypothesis": "testable prediction with numbers",
  "methodology": "minimal experiment",
  "falsifiability": "what result would disprove it",
  "bottleneckId": 6, "topicId": 16
}
```

Hypotheses are public and attributed. Only submit what the user actually concluded.

## synapse.get_3d_map

`{}` → nodes with PCA coordinates, mastery percentage, and connections. Use it to answer "what have
I covered, and what is adjacent?" — not as a progress metric on its own.
