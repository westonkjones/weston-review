# Lens: tests

Would the tests fail if this change were broken?

- For each behaviour the change adds or alters, find a test that exercises it. List any new branch, error path or edge case with no test.
- Look for tests that can't fail: assertions on mocks of the thing under test, missing assertions, assertions that only restate the fixture, snapshot tests updated blindly, `expect(true)` style assertions, and swallowed exceptions in tests.
- Tests that pass for the wrong reason: over-mocking that bypasses the real code, a test fixture that never reaches the changed path, time or ordering dependencies.
- Changed tests: an assertion weakened, a test deleted or skipped, or an expected value edited to match new behaviour without the spec supporting it.
- Test quality counts the same as production code: shared mutable fixtures, flakiness sources (sleep, real network, real clock).

At depth `deep`, run the relevant tests. Where it's cheap, show that a test fails if you mentally revert the changed line.

Missing coverage is a `suggestion` unless the untested path is the one a spec criterion depends on.
