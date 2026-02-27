how can I ensure copilot generates both positive and negative tests with JUnit via an agents.md?# Testing rules (JUnit)

You can “force” the behavior by making your `agents.md` **explicitly require a test matrix** (happy path + failure path + boundaries) and by telling the agent to **list the cases first** and only then write JUnit tests. Copilot agents tend to comply well when the requirements are measurable (counts, checklists, specific patterns like `assertThrows`, parameterized tests, etc.).

Here’s a **copy/paste** section you can add to your repo-level `agents.md` (or wherever Copilot Agent instructions live):

```md
# Testing rules (JUnit)

When you create or update production code, you MUST also create/update JUnit 5 tests.

## Definition of done for tests
For every public method (and any non-trivial package-private method):
1) **Positive coverage**: at least **1 happy-path** test proving the expected result.
2) **Negative coverage**: at least **1 failure-path** test that asserts:
   - thrown exception type (use `assertThrows`)
   - and (when meaningful) exception message or error code
3) **Boundary/edge coverage**: at least **1 edge case** (e.g., min/max, empty, null, whitespace, zero, overflow, single-element list).

If a method has input validation, negative tests must include:
- `null` (when applicable)
- empty/blank strings
- out-of-range numbers
- invalid enum/format values (if parsing)

## Process (do this before writing tests)
Before writing any test code, output a short "Test Plan" listing:
- Happy path case(s)
- Negative case(s)
- Boundary/edge case(s)
Include expected outcomes for each.

## JUnit 5 style requirements
- Use JUnit 5 (`org.junit.jupiter.*`)
- Prefer AssertJ assertions if available; otherwise use JUnit assertions.
- For negative tests, use `assertThrows(...)` and assert message when stable.
- Prefer `@ParameterizedTest` with `@ValueSource` / `@MethodSource` for multiple invalid inputs.
- Use Arrange / Act / Assert structure.
- Test names must describe behavior:
  `methodName_whenCondition_thenExpectedResult`

## Isolation and mocking
- Do NOT hit network/DB/filesystem in unit tests.
- Mock external dependencies with Mockito.
- Verify side-effects for negative tests when relevant (e.g., repository.save NOT called).

## Minimum expected output per method
At least **3 tests per method** (positive + negative + edge),
unless you justify in a comment why fewer are sufficient.
```

### A couple tweaks that make this “stick” even more

* Add a short **prompt header** to relevant modules too (e.g., in `/src/test/java/README.md`) repeating the “3 tests per method” rule.
* If you want to be *really* strict, add wording like: **“Do not consider the task complete until negative tests exist and pass.”**
* If you’re using SonarQube, add one more line: **“Tests must include assertions; no empty tests; no unnecessary `Thread.sleep`; no flaky time-based assertions.”**

If you share a small example method (or a class) you want Copilot to test, I can tailor the `Test Plan` template (happy/negative/edge) to your exact patterns (validation style, exceptions, Result/Either, Optional, etc.).
u justify in a comment why fewer are sufficient.