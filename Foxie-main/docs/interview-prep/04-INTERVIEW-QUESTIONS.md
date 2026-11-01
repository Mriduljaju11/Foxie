# 04 — Interview Questions and Rehearsal Answers

Use these as prompts, not scripts. Keep the first answer concise, then expand with source/code details when asked. Be accurate about what is implemented today versus what is described as future work.

## 30-second project pitch

> Foxie is a Python tool that scaffolds FastAPI CRUD features. A Typer-based CLI collects the project details and sends them to a FastAPI backend. The backend uses Gemini with file-based code examples to generate resource-specific CRUD code, and uses Jinja2 templates for standardized authentication files when requested. The CLI writes the returned files and generates the new project's configuration. I focused on separating the user workflow from generation and combining flexible AI output with more repeatable templates.

Personalize the final sentence to your own contribution. Avoid saying “I built” for work you did not personally do.

## Likely technical questions

### 1. Walk me through one scaffold request.

**Answer outline:** The CLI collects project/resource/fields/database/auth settings, resolves the Gemini key, POSTs JSON to `/scaffold`, and validates the response. The backend validates the request model and calls `generate_crud_feature()`. That parses fields, loads relevant `.example` snippets, calls Gemini for core files, optionally adds auth files from Jinja2 templates, and returns a `GeneratedCode` bundle. The CLI writes the bundle, formats Python files when Black is available, creates `pyproject.toml` and `.env`, then prints setup steps.

### 2. Why separate the CLI and backend?

**Answer outline:** The CLI focuses on interaction and local filesystem operations; the backend owns generation/provider integration. The separation allows the CLI and generation service to be packaged or deployed independently and provides a clear HTTP contract. The cost is network failure handling, API compatibility, and the need to protect credentials passed between components.

### 3. What does RAG mean in this project?

**Answer outline:** It loads selected FastAPI example files and includes their contents as context in the model prompt. Selection depends on database/auth settings. There is no vector database or semantic embedding search in this current path, so I would describe it as file-based example retrieval/context injection rather than a full vector RAG system.

### 4. Why use both Gemini and templates?

**Answer outline:** CRUD varies with user-defined resources and fields, so model generation is flexible. Authentication is more standardized, so templates make that portion faster and more consistent and avoid another model call. The main challenge is keeping template interfaces aligned with the generated router/models and testing both database variants.

### 5. Does structured JSON output make generated code correct?

**Answer outline:** No. It helps constrain the response to the expected `files` structure. It does not prove that the Python parses, imports, works with the selected database, or meets security requirements. Syntax checks, static checks, isolated runtime tests, and generated-project integration tests are still needed.

### 6. How are transient model failures handled?

**Answer outline:** The core generation function retries errors identified as 503/unavailable/overloaded up to three total attempts, waiting 2 then 4 seconds. It raises other errors. A next iteration could use typed provider error categories, bounded exponential backoff with jitter, metrics, and clearer retryable/non-retryable classification.

### 7. How do you handle API keys?

**Answer outline:** The CLI checks environment/dotenv sources, can prompt interactively, and can save the key in a user config file. It sends the key in the scaffold request; the backend passes it to the Gemini client rather than using a shared configured provider key. For production, require HTTPS remotely, prevent body logging, keep secrets out of errors, consider an OS-backed secret store, and document the plaintext local-config trade-off.

### 8. What would you improve first?

**Answer outline:** I would first add validation and tests around generated output: validate identifiers and field types, generate into a temporary directory, run Python syntax/import checks and database-specific tests, and only then promote the output to the requested directory. This addresses the largest correctness gap while making failures easier to explain to users.

### 9. What scalability issue do you see?

**Answer outline:** The FastAPI route is async but invokes synchronous generation code, including synchronous model calls and sleeps. Under concurrent requests, this can block the event loop. I would move generation to a worker/job queue or use a properly asynchronous provider client and return job status for long-running requests. I would also add rate limits, quotas, request IDs, and concurrency controls.

### 10. How would you test the system?

**Answer outline:** Unit-test field parsing, prompt construction, template rendering, and CLI response handling. Mock the Gemini SDK to test normal output, malformed output, and retry cases. Use FastAPI's test client for endpoint contracts. Generate sample SQL/MongoDB projects in temporary directories, run syntax/import checks, then execute integration tests against disposable databases. Avoid requiring a paid live provider key in ordinary CI.

### 11. What happens if file writing fails halfway through?

**Answer outline:** The current writer reports per-file OS errors and continues, so partial output is possible. I would write into a temporary directory, record per-file outcomes, validate the entire project, and atomically rename/promote only on success. I would also validate all returned paths stay under the chosen output root.

### 12. What is the difference between formatting and validation?

**Answer outline:** Black reformats Python source. It does not confirm syntax-independent correctness, type correctness, valid imports, database behavior, security, or API semantics. Formatting is one quality step, not a test suite.

## Be ready to discuss limitations honestly

- Current backend routes are `/scaffold` and `/health`.
- The current CLI supports SQL/MongoDB selection and `--enable-auth`; do not claim a current ReAct flow or `--protect-routes` option based only on older docs.
- The backend's example loader is file-based and not vector search.
- Generated output is not automatically proven correct by the JSON response or Black formatting.
- Field parsing checks basic `name:type` structure, not a comprehensive schema/identifier policy.
- The global CLI API-key file is written as plaintext.
- The generation route's synchronous work can block the event loop.

## STAR story worksheet

Prepare one story using the STAR structure:

- **Situation:** What user/developer problem made scaffolding valuable?
- **Task:** What did you own personally? Be specific.
- **Action:** What design/implementation choices did you make, and why? Mention one trade-off.
- **Result:** What became easier or more reliable? Use a measured result only if you have evidence.
- **Learning:** What would you change after reviewing the limitations above?

Do not invent metrics. If you did not measure speed/cost savings, say that templates are intended to reduce model calls rather than claiming a verified percentage.

## Last-minute rehearsal checklist

- Can you explain the project in 30 seconds and 2 minutes?
- Can you draw the request flow from CLI to generated files?
- Can you point to the source file for each major responsibility?
- Can you explain why example-context loading is not vector search?
- Can you name one real limitation and a practical improvement?
- Can you distinguish implemented behavior from roadmap/documentation claims?
- Can you explain your own contribution without overstating it?
