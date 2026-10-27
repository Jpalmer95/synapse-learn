# Course authoring contract

What the ingest endpoint accepts, and what the quality gate rejects. Uploading is free (no platform
AI); the gate is the only thing standing between your payload and the public catalogue.

## Payload

```json
{
  "topicId": 16,
  "topic": { "title": "Electromagnetism", "description": "One or two concrete sentences." },
  "categoryId": 5,
  "learningIntent": "standard",
  "archetype": "focused_skill",
  "goalText": "Optional: one sentence on what the learner wants",
  "createGoal": true,
  "kind": "core",
  "plan": {
    "contentType": "formula_heavy",
    "scope": "focused",
    "rationale": "why this shape",
    "recommendedOER": [{ "name": "OpenStax UP2", "url": "https://openstax.org/books/university-physics-volume-2/pages/1-introduction", "reason": "spine text" }]
  },
  "units": [
    {
      "title": "Specific title, not 'Introduction to X'",
      "difficulty": "beginner",
      "outline": "What the learner can do afterwards.",
      "contentJson": {
        "concept": "## First section\n\n…\n\n## Second section\n\n…",
        "keyTakeaways": ["4 usable claims"],
        "analogy": "One tight analogy.",
        "example": { "title": "Worked example", "content": "Real numbers and units.", "code": "" },
        "quiz": [{ "question": "…", "options": ["4"], "correctIndex": 0, "explanation": "why" }],
        "externalResources": [{ "title": "…", "url": "https://…", "type": "book", "description": "why this one" }],
        "crossLinks": [],
        "mermaidDiagram": "graph LR\n  A[X] --> B[Y]"
      }
    }
  ]
}
```

`units` max 80. Options that change behaviour:

| field | effect |
|---|---|
| `topicId` | append to an existing course instead of creating one (modular add-on) |
| `archetype` | fixes unit count and section budget; persisted in `planJson` |
| `learningIntent` | `standard`/`deep` → full view; `speed_run`/`survey` → skim view |
| `kind` | `core` \| `niche` \| `jit`; JIT/niche stays private unless published later |
| `force` | admins only: bypass the gate (recorded in the quality report) |

## What the gate rejects (422, with a per-unit report)

1. **Template prose.** Any of: "this is where bugs hide", "the classic trap", "deceptively simple
   idea", "your gateway into", "this is the workhorse of practical", "implementation pattern",
   "bridges theory and practice", "in code reviews, this is where", "study tip".
2. **Repeated quiz questions.** Duplicate stems inside a unit are errors; the same stem repeated
   across many units fails the course.
3. **Placeholder resources.** `?page_search_query=`, `khanacademy.org/search`, `google.com/search`,
   `wikipedia.org/wiki/Main_Page`, `example.com`, bare domain roots.
4. **Thin prose.** Below ~60% of the archetype's section budget.
5. **Few resources / takeaways / quiz items** compared to the archetype budget.

Warnings (accepted, but recorded and shown in the admin report): unrecognised resource host,
repeated URL within a course, no worked example, no analogy, prose well above the budget.

## Section budget by archetype

| archetype | units | words/section | sections/unit | takeaways | resources | worked example |
|---|---|---|---|---|---|---|
| rapid_bootstrap | 3–5 | 140–240 | 2 | 3 | 2–3 | optional |
| focused_skill | 6–10 | 200–320 | 2 | 4 | 2–4 | yes |
| foundation | 18–28 | 420–650 | 2–3 | 4 | 3–5 | yes |
| deep_dive | 12–18 | 350–520 | 2–3 | 4 | 3–5 | yes |
| frontier_survey | 6–10 | 260–380 | 2 | 3 | 3–5 | sometimes |
| cross_domain | 8–14 | 320–480 | 2–3 | 4 | 3–5 | yes |

## Resource rules

Curated means *specific*: a chapter URL, a specific OCW course, a named paper, a real video. Trusted
hosts include openstax.org, ocw.mit.edu, libretexts.org, en.wikipedia.org (article, not search),
feynmanlectures.caltech.edu, arxiv.org, plato.stanford.edu, developer.mozilla.org, docs.python.org,
postgresql.org. Anything else is accepted with a warning but reviewed by a human before publishing.

Do not invent arXiv IDs, DOIs, or video IDs. Omit a number you are unsure of rather than guess it.

## Publishing

- New courses: `published = false`, visible only to the author.
- `quality.grade == "ok"` + review → published to the commons.
- Published courses are the shared corpus: anyone can take them, extend them with add-ons, or fork a
  niche variant. They are not regenerated per user.
- Course text: CC BY-SA 4.0. Attribution stays on the course record.
