---
name: ignore-tests-teku
description: Work on the Teku codebase without running tests or reading test code beyond what is needed to keep it compiling. Use when the user says "ignore tests", "no tests", "skip tests", "don't run tests", or invokes /ignore-tests-teku. Keeps test sources and test output out of the context, never runs tests, and only touches test code to fix compilation errors caused by production changes.
---

# Ignore tests (Teku)

The user wants this session focused on production code. Tests cost time (Gradle test runs
take minutes) and context (test classes are long and repetitive). The one exception is
compilation: the whole project, tests included, must still compile after every change.
Until the user says otherwise:

## Do not run tests

Never run any of these, in the foreground or in the background:

- `./gradlew test`, `:<module>:test`, `--tests ...`
- `integrationTest`, `referenceTest`, `acceptanceTest`, `propertyTest`, `jmh`
- `./gradlew build` and `check` (they run tests); `ManualReferenceTestRunner`
- `convertRefTests`, `expandRefTests`, `generateReferenceTestClasses`, `compileReferenceTestJava`
  (reference test setup downloads and generates fixtures)

## Do not load test code

Do not Read, `cat`, `sed -n`, `head` or otherwise open files under these paths, except as
described in [Keep tests compiling](#keep-tests-compiling):

| Path | What it is |
|---|---|
| `**/src/test/**` | unit tests |
| `**/src/testFixtures/**` | shared test helpers (`DataStructureUtil`, `StorageSystem`, `ChainBuilder`, ...) |
| `**/src/integration-test/**` | integration tests |
| `**/src/property-test/**` | property tests |
| `**/src/jmh/**` | benchmarks |
| `**/src/referenceTest/**`, `eth-reference-tests/` | consensus spec reference tests |
| `acceptance-tests/`, `eth-benchmark-tests/`, `eth-tests/`, `fork-choice-tests/`, `fuzz/` | test-only modules |
| `**/build/test-results/**`, `**/build/reports/tests/**` | test output |

When searching, exclude them so test hits do not flood the results:

- Grep tool: `glob: "**/src/main/**"` (or `"!**/src/{test,testFixtures,integration-test,property-test,jmh,referenceTest}/**"`)
- Shell: `grep -r --include='*.java' --exclude-dir={test,testFixtures,integration-test,property-test,jmh,referenceTest} ...`
- Glob tool: patterns rooted at `**/src/main/java/**`

If a usage only shows up in tests, report the file path and line from the search output. Do not open the file.

## Do not write tests

Do not add new tests and do not change what existing tests check. No new test methods, no
edited assertions or expected values, no deleted or `@Disabled` tests. If a production change
probably alters test behaviour, list the likely affected test files instead (found with a
filename-only search such as `Glob **/src/test/**/FooTest.java`).

## Keep tests compiling

After production code compiles, compile the test source sets of the modules you changed
**and of every module that depends on them** (a changed signature in `ethereum:spec` breaks
tests all over the repo). When in doubt, compile everything:

```bash
./gradlew compileJava compileTestFixturesJava compileTestJava compileIntegrationTestJava compilePropertyTestJava compileJmhJava
```

Or per module, e.g. `./gradlew :ethereum:statetransition:compileTestJava`. Module paths
follow the folders, so `networking/eth2` is `:networking:eth2`.

If test compilation fails because of your change, fix it with minimal context:

1. Take the file and line from each compiler error. Use `--quiet` or tail the output, since
   only the errors are needed.
2. Read only a small window around each error (about 20 lines, using `offset`/`limit`),
   never the whole test class. Read more only if the fix needs it, such as the enclosing
   method or the imports block.
3. Make the smallest mechanical fix that restores compilation: update a call to a changed
   signature, pass a new constructor argument (prefer an existing test helper or a
   `mock(...)`), rename a method or type, fix an import, or adjust a type.
4. Keep the test's intent. If the only way to compile is to change what the test asserts,
   or to remove the test, stop and tell the user rather than doing it.
5. Recompile and repeat until clean. Fix `testFixtures` errors first, because they cascade
   into the test source sets of every module that uses them.

Fixing compilation is not permission to run the tests you touched.

## Production verification

```bash
./gradlew :<module>:compileJava          # e.g. :ethereum:statetransition:compileJava
./gradlew compileJava                    # everything, when the change spans modules
./gradlew spotlessApply                  # formatting, also formats test files you fixed
```

## Wrapping up

End each change with a short note that says:

- tests were not run
- which test files were edited only to fix compilation
- which test files probably need behavioural updates, so the user can run or fix them later

## Turning it off

Stop following this skill once the user asks to run, read or write a specific test. Treat
that as lifting the restriction for that request only, unless they say it applies to the rest
of the session.
