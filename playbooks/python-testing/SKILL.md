---
name: python-testing
description: Use when writing or fixing Python tests with pytest, including fixtures, parametrization, mocks, async tests and FastAPI endpoints.
license: MIT
compatibility: Python 3.9 or later with pytest. Checked on 2026-10-13 with pytest 9.1, pytest-asyncio 1.4, FastAPI 0.143 and httpx 0.28.
metadata:
  status: "candidate"
  topic: "testing"
  created: "2026-10-13"
  updated: "2026-10-13"
  verified: "2026-10-13"
  origin: "ecc"
  origin-url: "https://github.com/affaan-m/ECC"
  origin-commit: "b92f181b3b748d1abbc2afd94af6aa7d8c8fca29"
  origin-path: "skills/python-testing/SKILL.md"
  approval: "none"
---

# Python testing with pytest

Adapted from the upstream skill and cut down to the decisions that go wrong in practice. Basic pytest syntax is assumed.

## Steps

1. **Find out how this project runs its tests before writing any.** Read `pyproject.toml`, `pytest.ini`, `setup.cfg` or `tox.ini` for `testpaths`, markers, `addopts` and the async mode. Run the existing suite once and note what passes. If a plugin the test needs is missing (pytest-asyncio, pytest-cov, httpx), say so and ask before adding it.
2. **Put the test where the project puts tests,** in a file named `test_*.py`. Shared fixtures go in the nearest `conftest.py`. A fixture used by one file stays in that file.
3. **Test behaviour.** One behaviour per test, with a name that states it (`test_login_with_wrong_password_returns_401`). Assert on return values, raised exceptions and visible side effects. Do not assert on private attributes or call order unless the call is the behaviour.
4. **Fixtures.**
   - Use `yield` for teardown. Code after `yield` runs even when the test fails.
   - The default scope is one test. Widen to `module` or `session` only for something expensive and read-only. A widened fixture that tests change is shared state.
   - Use `tmp_path` for files. `tmpdir` is the legacy form.
   - Use `monkeypatch` for environment variables, attributes and the working directory. It undoes itself.
5. **Parametrize instead of looping inside a test:** `@pytest.mark.parametrize("raw, expected", [...], ids=[...])`. Each case then fails under its own name.
6. **Exceptions:** `with pytest.raises(ValueError, match="..."):` around the one call that should raise, and nothing else inside the block.
7. **Mocks.**
   - Patch the name where it is looked up. For `from app.services import send` used in `app/orders.py`, patch `app.orders.send`. Patching `app.services.send` leaves the caller holding the real function.
   - Use `autospec=True` so a call with the wrong signature fails.
   - Mock at the boundary: network, clock, filesystem, paid API. Do not mock the unit under test or pure functions.
   - Async callables need `AsyncMock`. Assert with `assert_awaited_once_with`.
8. **Async tests.** Check the configured mode first. pytest-asyncio defaults to strict mode, where each async test needs `@pytest.mark.asyncio` and async fixtures need `@pytest_asyncio.fixture`. With `asyncio_mode = "auto"` in the config, plain `async def test_...` works.
9. **FastAPI endpoints.**
   - Synchronous tests: `from fastapi.testclient import TestClient`, then `client = TestClient(app)`. `response.json()` is a method call.
   - Replace a dependency with `app.dependency_overrides[get_db] = fake_db` and clear the dict in fixture teardown.
   - Startup and shutdown code (the lifespan) runs only when the client is used as a context manager: `with TestClient(app) as client:`.
   - Async tests: `httpx.AsyncClient(transport=httpx.ASGITransport(app=app), base_url="http://test")`.
10. **Run narrow, then wide.** `pytest path/to/test_file.py::test_name -x` while working, the whole suite before reporting.

## Verify

- The new test fails when the behaviour is broken. Break the code or invert the assertion once and watch it fail. A test never seen failing proves nothing.
- The whole suite passes. Quote the command and the pass count in the report.
- No test depends on another test's order or leftovers. Run the new file alone and inside the full suite.
- Unit tests touch no real network, clock or user files.
- Coverage is a by-product. Report it if the project measures it. Do not write tests without assertions to raise it.

## Pitfalls

- The upstream text shows `app.test_client()` and `response.json["id"]` under a heading that names FastAPI. That is Flask's interface. FastAPI uses `TestClient(app)` and `response.json()`.
- An async test without the marker fails in strict mode under pytest 9 ("async def functions are not natively supported"). Older versions skipped it with a warning, which looked like a pass.
- `@patch` decorators apply bottom-up: the lowest decorator is the first mock argument.
- A `session`-scoped fixture holding a database connection makes tests pass alone and fail together.
- `--disable-warnings` in `addopts`, as upstream suggests, hides the deprecation warnings that explain the next breakage. Leave warnings visible.
- With FastAPI 0.143 the test client prints a deprecation warning that names `httpx2` as the replacement for `httpx` (seen 2026-10-13). Check the project's pinned versions before changing a dependency because of it.
- Upstream sets a coverage floor of 80%. That number is the project's decision. Use the project's own if it has one.
