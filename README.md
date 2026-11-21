# ignore-tests-teku-skill

An agent skill for working on [Teku](https://github.com/Consensys/teku) without running tests or loading test code into the agent's context.

Teku's test suites take minutes to run, and the test classes are long, so reading them uses up a lot of the agent's context. For many tasks, such as exploring the code, prototyping, reviewing production logic or making quick fixes, you don't need either. With this skill active, the agent:

- **does not run tests**: no `test`, `integrationTest`, `referenceTest`, `acceptanceTest`, `propertyTest`, `jmh`, `build` or `check`
- **does not read test code**: it skips `src/test`, `src/testFixtures`, `src/integration-test`, `src/property-test`, `src/jmh`, `src/referenceTest`, the test-only modules and `build/` test output, and leaves them out of searches
- **does not write tests**: it lists the test files a change probably affects instead of editing them
- **checks changes by compiling production code only**: `./gradlew :<module>:compileJava`, plus `spotlessApply`
- **ends each change with a short note** that tests were not run and which test files may need attention

## Installation

The skill is a single `SKILL.md` file. Put it in a folder named `ignore-tests-teku` wherever your agent loads skills from.

### Claude Code: all projects

```bash
git clone https://github.com/StefanBratanov/ignore-tests-teku-skill.git ~/.claude/skills/ignore-tests-teku
```

On Windows (PowerShell):

```powershell
git clone https://github.com/StefanBratanov/ignore-tests-teku-skill.git "$env:USERPROFILE\.claude\skills\ignore-tests-teku"
```

### Claude Code: one Teku checkout

From the root of your Teku clone:

```bash
git clone https://github.com/StefanBratanov/ignore-tests-teku-skill.git .claude/skills/ignore-tests-teku
```

Teku's `.gitignore` already ignores `.claude/`, so the skill won't show up in your commits.

### Other agents

Any agent that supports the `SKILL.md` format (YAML frontmatter with `name` and `description`) can use it. Copy the folder into that agent's skills directory. For agents without skill support, paste the body of `SKILL.md` into the agent's instructions file, such as `AGENTS.md`, or into the conversation.

### Updating

```bash
git -C ~/.claude/skills/ignore-tests-teku pull
```

## Usage

Start a session in your Teku checkout, then either:

- invoke it directly: `/ignore-tests-teku`
- or ask in plain language, for example "ignore tests", "skip tests" or "don't run tests". The skill's description makes the agent load it automatically.

Then work as usual:

```
> /ignore-tests-teku
> Add a debug log when ForkChoice rejects an execution payload envelope
```

The agent edits `src/main`, runs `./gradlew :ethereum:statetransition:compileJava`, and ends with something like:

> Tests not run. Likely affected: `ethereum/statetransition/src/test/java/.../ForkChoiceTest.java`

### Lifting the restriction

If you ask the agent to run, read or write a specific test, it does so for that request only. Say so explicitly if you want test work back for the rest of the session ("you can run tests again from now on"). A new session starts without the skill unless you invoke it again.

## Limitations

- A skill is guidance, not enforcement. The agent can still slip, for example by opening a test file that shows up in a stack trace. If you need a hard guarantee, add a `PreToolUse` hook in Claude Code that rejects file reads and shell commands matching the test paths.
- If a change breaks test compilation, compiling only `compileJava` won't catch it. Run `./gradlew compileTestJava` or the relevant tests yourself before opening a PR.
- The path list matches Teku's current layout. If new test source sets or modules are added, update the tables in `SKILL.md`.
