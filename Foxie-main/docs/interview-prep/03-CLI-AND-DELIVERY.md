# 03 — CLI, Output Files, and Delivery

## CLI responsibilities

The main CLI implementation is [cli.py](../../foxie-cli/src/foxie_cli/cli.py). It uses Typer for commands/options and Rich for prompts and terminal feedback. The package entry point exposes the `foxie` command.

The main scaffold command is:

```text
foxie scaffold fastapi-crud
```

The CLI supports interactive input and command-line options for project name, resource, fields, database type, and `--enable-auth`. In command mode, project name, resource, and fields are required; database type defaults to SQL. The current CLI source does not define `--protect-routes` or a ReAct-mode option.

## API key and backend URL

- The key is read from the process environment or dotenv files; if unavailable, the CLI prompts for it.
- `foxie config` saves a key to `~/.config/foxie/config.env` for reuse.
- The CLI sends the key in the JSON body of each scaffold request.
- `FOXIE_BACKEND_URL` overrides the configured backend URL.
- `FOXIE_SCAFFOLD_TIMEOUT` controls the request timeout; the default is 300 seconds.

Security discussion: saving the key in a plaintext config file is convenient but has a local-at-rest risk. Remote requests should use HTTPS, backend/proxy logs should not capture request bodies, and errors should never echo credentials.

## CLI request and response path

1. Collect and validate required CLI options.
2. Resolve the API key.
3. Build JSON containing `project_name`, `resource`, `fields_str`, `database_type`, `enable_auth`, and `api_key`.
4. POST to `<backend URL>/scaffold` using `requests` with the configured timeout.
5. Check HTTP status and content type, decode JSON, and construct the CLI's `GeneratedCode` model.
6. Handle connection, timeout, HTTP, invalid JSON, and model-validation failures with terminal messages and a non-zero exit.
7. Write returned files, generate project configuration, and print setup instructions.

The CLI models mirror the backend's generated-file shape. See [models.py](../../foxie-cli/src/foxie_cli/core/models.py).

## How files are written

[file_writer.py](../../foxie-cli/src/foxie_cli/utils/file_writer.py) provides the writing helpers:

- Creates directories under the selected project output directory.
- Removes an outer Markdown code fence if one is present.
- Writes each file as UTF-8.
- Attempts to format `.py` files by invoking Black with the active Python interpreter.
- Reports formatting failures, but does not stop successful file writes just because formatting failed.

The CLI then uses [template_generator.py](../../foxie-cli/src/foxie_cli/utils/template_generator.py) to create:

- `pyproject.toml`, with dependencies selected for SQL or MongoDB and optional auth.
- `.env`, with database and security defaults.

Interview caveat: formatting is not validation. Also, the generated `.env` contains a placeholder `SECRET_KEY`; it must be changed before real deployment. Inspect generated files before running or shipping them.

## Packaging and running locally

The CLI package uses a `src/` layout and declares the `foxie` console script in its project metadata. The backend and CLI have separate Python project files and Dockerfiles. At repository root, Compose defines the backend and CLI services and mounts source/data for backend development.

For a local walkthrough, study these sources:

- [CLI package metadata](../../foxie-cli/pyproject.toml)
- [Backend package metadata](../../foxie-backend/pyproject.toml)
- [Compose configuration](../../docker-compose.yml)
- [CLI file writer](../../foxie-cli/src/foxie_cli/utils/file_writer.py)

The CLI defaults to a hosted backend URL, while local development can point it at `http://127.0.0.1:8000` through `FOXIE_BACKEND_URL`. Inside Docker Compose, the backend is addressable by its service name on the Compose network.

## Failure handling worth explaining

- **Connection error:** backend unreachable or URL/network problem.
- **Timeout:** generation exceeded the client timeout; the user can raise the timeout, though the CLI currently does not resume a request.
- **HTTP error:** the CLI reports the status and tries to avoid printing an HTML error page as if it were useful JSON.
- **Invalid response:** JSON decoding or Pydantic response parsing fails.
- **Filesystem error:** the CLI reports a write failure; the current writer handles per-file OS errors by reporting and continuing.
- **Formatter failure:** file remains written, but the CLI warns that formatting failed.

A stronger future design could make writes transactional (write to a temporary output directory, validate, then promote), detect path traversal in backend-returned paths, and summarize failed files distinctly from successful ones.

## Current documentation mismatches to avoid in an interview

The implementation is the source of truth. Current source inspection shows:

- Backend routes: `/scaffold` and `/health` only.
- CLI auth flag: `--enable-auth`; no `--protect-routes` flag currently exists.
- No ReAct endpoint or CLI mode is wired into the current `main.py`/`cli.py` path.
- The backend expects each request to carry the API key. A Compose environment variable for `GOOGLE_API_KEY` should not be described as the active key path without changing the implementation.

Some older feature/structure docs mention these options as if they were active. Phrase unimplemented roadmap items as future work, not completed functionality.

## Questions to practice

1. What happens when a developer runs the command without flags?
2. How is the API key resolved and how does it reach the backend?
3. What kinds of network and response errors does the CLI handle?
4. What steps happen after the JSON response arrives?
5. What security and filesystem protections would you add before production use?
