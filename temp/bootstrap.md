---
description: Detect project runtime and write docs/agent-runtime.md
---

# Bootstrap

Detect the project's runtime configuration and write `docs/agent-runtime.md`. This file is consumed by implementation subagents to know how to run and test the project.

This command is also auto-triggered by `/implement` when `docs/agent-runtime.md` is missing. Run it manually to regenerate or reset the file.

## Process

**Step 1 — Discover runtime info**

Search in this order, stopping at the first source that provides useful information:

1. Read `CLAUDE.md` — look for test commands, run commands, package manager
2. Read `AGENT.md` — same
3. Read `README.md` — look for "getting started", "running tests", "development" sections

**Step 2 — Infer from project structure**

Also inspect the project root for these files to confirm or fill gaps:

| File | Inferred runtime |
|---|---|
| `gradlew` | Gradle — `./gradlew test`, `./gradlew bootRun` |
| `package.json` | Node — check `scripts.test` and `scripts.start` |
| `Makefile` | Make — look for `test` and `run` targets |
| `pyproject.toml` or `setup.py` | Python — `pytest` or `python -m pytest` |
| `pom.xml` | Maven — `mvn test`, `mvn spring-boot:run` |
| `Cargo.toml` | Rust — `cargo test`, `cargo run` |
| `go.mod` | Go — `go test ./...`, `go run .` |

**Step 3 — Write the file**

Create `docs/` directory if it doesn't exist. Write `docs/agent-runtime.md` with this exact schema:

```
## Runtime

test_command: [full command to run all tests]
run_command: [full command to start the app]
stop_command: [command to stop the app, if applicable]
test_scope_flag: [flag to scope tests, e.g. --tests {classname} or -k {name}]
package_manager: [gradle | maven | npm | yarn | pnpm | pip | cargo | go | make]
notes: [any relevant context, e.g. "Spring Boot 3.x, requires Docker for tests"]
```

**Step 4 — Handle unknowns**

If you cannot determine a value with confidence, leave it empty rather than guessing. After writing:

- If all fields are populated: report success and show the file path
- If any field is empty: warn the user with a clear message listing which fields need manual input:
> ⚠️ `docs/agent-runtime.md` written, but the following fields could not be determined automatically:
> - [list of empty fields]
> Please fill them in before running `/implement`.
