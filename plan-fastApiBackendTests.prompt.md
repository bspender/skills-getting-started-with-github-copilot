## Plan: Add FastAPI Backend Tests

Add an isolated pytest suite under a new top-level `tests/` directory. Reuse FastAPI's `TestClient`, restore the module-level `activities` mapping around every test, and structure each test explicitly with Arrange, Act, and Assert sections. Lock down the current redirect, listing, signup, unregister, validation, and error contracts without changing application behavior.

**Steps**
1. Add an explicit `pytest` entry to `/workspaces/skills-getting-started-with-github-copilot/requirements.txt` so `pip install -r requirements.txt` installs the required test runner; use pytest fixtures, test discovery, and assertions throughout the suite, and keep existing dependencies unchanged.
2. Create `/workspaces/skills-getting-started-with-github-copilot/tests/conftest.py` with a session-appropriate `TestClient` fixture for `src.app.app` and an autouse fixture that deep-copies `src.app.activities`, then restores it in place after each test. Keep reusable infrastructure and cleanup in fixtures; keep scenario-specific inputs and state changes in each test's Arrange section.
3. Create `/workspaces/skills-getting-started-with-github-copilot/tests/test_app.py`. Give every test three visually distinct sections: Arrange defines route inputs, expected payloads, and any fixture-local activity state; Act performs exactly one client request; Assert checks the response and relevant application state. Use concise `# Arrange`, `# Act`, and `# Assert` comments consistently.
4. Test `root` by arranging a client configured not to follow redirects, acting with `GET /`, then asserting the `307` response and `/static/index.html` location.
5. Test `get_activities` by arranging the expected seeded names and public field set, acting with `GET /activities`, then asserting `200`, the names, and each activity's fields (`description`, `schedule`, `max_participants`, `participants`) without duplicating every seed value.
6. Cover `signup_for_activity` with separate AAA tests for successful mutation and exact success message, unknown activity (`404`), duplicate participant (`400`), full activity (`400`, arranged through fixture-local state), invalid email (`422`), and missing email (`422`). Assertions must include participant-list mutation on success and non-mutation on failures.
7. Cover `unregister_from_activity` with separate AAA tests for successful removal and exact success message, unknown activity (`404`), missing participant (`404`), and missing email (`422`). Assertions must include removal on success and unchanged state on failures. The DELETE route currently accepts `str`, so invalid-email-format rejection is deliberately not expected.
8. Run the focused suite and then the repository-wide pytest command to confirm discovery through the existing `/workspaces/skills-getting-started-with-github-copilot/pytest.ini` configuration.

**Relevant files**
- `/workspaces/skills-getting-started-with-github-copilot/src/app.py` — existing `app`, `activities`, `root`, `get_activities`, `signup_for_activity`, and `unregister_from_activity` contracts under test; no planned edits.
- `/workspaces/skills-getting-started-with-github-copilot/tests/conftest.py` — new shared client and in-memory state isolation fixtures.
- `/workspaces/skills-getting-started-with-github-copilot/tests/test_app.py` — new endpoint and validation regression tests.
- `/workspaces/skills-getting-started-with-github-copilot/requirements.txt` — add the missing `pytest` test dependency.
- `/workspaces/skills-getting-started-with-github-copilot/pytest.ini` — reuse existing `pythonpath = .`; no planned edit.

**Verification**
1. Run `pip install -r requirements.txt` from the workspace root and confirm the `pytest` command is available from the declared project dependencies.
2. Run `pytest tests/test_app.py -v` for focused endpoint verification.
3. Run `pytest -v` to confirm normal discovery and no broader regressions.
4. Repeat the suite to confirm mutable activity state is restored between cases.

**Decisions**
- Scope is backend regression tests only; no route behavior, response schema, or storage refactor is included.
- Use one route test module because the backend is currently one small application module; split files later if the API grows.
- Assert exact error details and success messages because they are current user-visible API contracts.
- Structure every endpoint scenario with consistent `# Arrange`, `# Act`, and `# Assert` markers; avoid hiding scenario-specific setup or assertions inside helper functions.
- Cover invalid email validation on POST only: `EmailStr` enforces it there, while DELETE intentionally remains a plain string in current production code.
- Coverage tooling, frontend tests, asynchronous clients, and CI workflow changes are excluded.
