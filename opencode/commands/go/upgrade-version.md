---
description: Uprage Golang version, test the code and builds.
---

Update the Go version in this project to $1.

Steps:
1. Find all files that reference the current Go version:
   - go.mod — update the go directive
   - Dockerfile / docker-compose.yml — update FROM golang:... image tags
   - .github/workflows/*.yml — update go-version: fields
   - .gitlab-ci.yml - Update MMA_GOLANG_VERSION and all related variables 
   - Any other config files referencing a Go version (.tool-versions, Makefile, etc.)
2. Replace every occurrence of the old Go version with $1.
3. Run go mod tidy to ensure module compatibility.
4. Run the full test suite with go test ./... and report results.
5. If tests fail, investigate the cause — do not revert the version change unless the failure is directly caused by an incompatibility with Go $1.
6. Docker image validation — skip this entire step if no Dockerfiles are found in the project:
   - Find all Dockerfiles in the project (e.g. Dockerfile, Dockerfile.development, Dockerfile.production, etc.)
   - For each Dockerfile found, run:
          docker build --no-cache -f <path/to/Dockerfile> -t app:validate-<suffix> .
        - After each successful build, immediately remove the image to keep the environment clean:
          docker rmi app:validate-<suffix>
        - Report success or failure per Dockerfile.
   - If a build fails due to a missing or inaccessible base image (e.g. private registry auth error), note it as "skipped — registry not accessible" and continue with remaining Dockerfiles.
   - Do not push any images.
Constraints:
- Only modify version strings, do not touch unrelated code.
- Do not change go.sum manually — let go mod tidy handle it.

Done when: all version references are updated to $1, go test ./... passes, all accessible Dockerfiles build successfully, and all validation images have been removed.
