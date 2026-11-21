---
name: ignore-tests-teku
description: Work on the Teku codebase without running tests or reading test code. Use when the user says "ignore tests", "no tests", "skip tests", "don't run tests", or invokes /ignore-tests-teku. Keeps test sources, test fixtures and test output out of the context and verifies changes by compiling production code only.
---

# Ignore tests (Teku)

The user wants this session focused on production code. Tests cost time (Gradle test runs
take minutes) and context (test classes are long and repetitive). Until the user says otherwise:

## Do not run tests

Never run any of these, in the foreground or in the background:

- `./gradlew test`, `:<module>:test`, `--tests ...`
- `integrationTest`, `referenceTest`, `acceptanceTest`, `propertyTest`, `jmh`
- `./gradlew build` and `check` (they run tests); `ManualReferenceTestRunner`
- `convertRefTests`, `expandRefTests`, `generateReferenceTestClasses` (reference test setup)

## Do not load test code

Do not Read, `cat`, `sed -n`, `head` or otherwise open files under these paths:

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

Do not add, update or delete test classes, even when a production change breaks a test's
compilation or behaviour. Instead, tell the user which tests are likely affected (by path,
found with a filename-only search such as `Glob **/src/test/**/FooTest.java`).

## How to verify instead

Compile only production source sets:

```bash
./gradlew :<module>:compileJava          # e.g. :ethereum:statetransition:compileJava
./gradlew compileJava                    # everything, when the change spans modules
./gradlew spotlessApply                  # formatting, safe to run
```

Do not run `compileTestJava`, `compileTestFixturesJava` or `compileIntegrationTestJava`.
Module paths follow the folders, e.g. `networking/eth2` is `:networking:eth2`.

Check compile output with `--quiet` or by tailing the last lines. Only the errors are needed.

## Wrapping up

End each change with one line saying tests were not run, plus any test files that probably
need updating, so the user can run or fix them later.

## Turning it off

Stop following this skill once the user asks to run, read or write a specific test. Treat
that as lifting the restriction for that request only, unless they say it applies to the rest
of the session.
