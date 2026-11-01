# Foxie Interview Preparation — Study Roadmap

This folder is a focused interview-preparation pack for the Foxie codebase. Study it in the order below: understand the system first, then trace a real request through the backend and CLI, and finish by rehearsing interview answers.

## Recommended order

| Order | Guide | What to learn |
|---|---|---|
| 1 | [01 — Architecture](01-ARCHITECTURE.md) | Product purpose, service boundaries, request/data flow, and design trade-offs |
| 2 | [02 — Backend and generation](02-BACKEND-GENERATION.md) | FastAPI contract, Gemini generation, example-context loading, auth templates, failure paths |
| 3 | [03 — CLI and delivery](03-CLI-AND-DELIVERY.md) | Typer commands, API-key resolution, HTTP handling, output writing, configuration, Docker |
| 4 | [04 — Interview questions](04-INTERVIEW-QUESTIONS.md) | Short project pitch, likely questions, honest limitations, and follow-up ideas |

Read the referenced source files while studying each guide. The links point to the implementation, so use the implementation as the source of truth when older docs disagree.

## A practical study plan

### Session 1 — Explain the product and architecture

1. Read this roadmap and [01 — Architecture](01-ARCHITECTURE.md).
2. Draw the CLI → backend → Gemini/template → generated-files flow from memory.
3. Practice a one-minute explanation of why the CLI and backend are separate.

### Session 2 — Trace generation end to end

1. Read [02 — Backend and generation](02-BACKEND-GENERATION.md).
2. Trace `POST /scaffold` through `generate_crud_feature()` and the response models.
3. Explain what “RAG” means here and how it differs from a vector-search system.

### Session 3 — Trace the user experience and delivery

1. Read [03 — CLI and delivery](03-CLI-AND-DELIVERY.md).
2. Follow the CLI from prompt/flags through the HTTP request to files on disk.
3. Explain how you would handle API-key safety, timeouts, partial output, and generated-code validation.

### Session 4 — Rehearse and verify

1. Read [04 — Interview questions](04-INTERVIEW-QUESTIONS.md).
2. Answer the questions aloud without reading, then check your answer against the guide and code.
3. Run a small local demo if the Gemini API key and dependencies are available. Never expose a real API key in a screen recording, commit, or interview notes.

## 30-minute quick review

- 5 minutes: project pitch and architecture diagram.
- 10 minutes: one request’s full path through the CLI and backend.
- 5 minutes: hybrid generation, prompt examples, and auth templates.
- 5 minutes: limitations and what you would improve next.
- 5 minutes: questions for the interviewer and a concise project demo.

## What to be ready to demonstrate

- The CLI accepts project name, resource, fields, database choice, and optional auth.
- The backend exposes `POST /scaffold` and `GET /health` in the current implementation.
- The backend returns a typed list of file paths, file contents, and descriptions.
- The CLI writes generated Python files and also creates a `pyproject.toml` and `.env`.
- The current implementation is a useful scaffold generator, not a guarantee that every generated project is production-ready. Review and test generated output before using it.

## Source of truth and documentation caveat

Some repository documentation describes ReAct generation and protected routes, but those features are not present in the current CLI/backend source inspected for this guide. The live backend routes are in [main.py](../../foxie-backend/app/main.py), and the CLI options/flow are in [cli.py](../../foxie-cli/src/foxie_cli/cli.py). Do not claim a feature is implemented unless you can point to its current code.
