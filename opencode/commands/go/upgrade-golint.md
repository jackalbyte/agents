---
description: Uprage golangci-lint version.
---

Upgrade golangci-lint to v2. Make these changes:
1. Run `golangci-lint migrate`
2. In .gitlab-ci.yml: add/update `MMA_GOLANG_CI_LINT_IMAGE: "v2.11-alpine"` variable; update the lint script to use `--output.code-climate.path stdout` instead of `--out-format code-climate`.
3. Commit changes using `/go/commit` command
4. Run the linter and fix any issues it reports.
5. Commit changes using `/go/commit` command
