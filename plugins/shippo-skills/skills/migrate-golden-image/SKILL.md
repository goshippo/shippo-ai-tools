---
name: migrate-golden-image
description: Migrate a Shippo microservice from custom Docker base image to standardized golden image with phased execution and human-in-the-loop gates
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git *, make *, aws ecr *, docker *, poetry *, go mod *, grep *), Task, WebSearch
---

# Migrate Microservice to Golden Images

## Context

You are migrating a Shippo microservice from a custom Docker base image to the standardized golden image. This microservice follows the py-zoo template structure (https://github.com/goshippo/py-zoo) and uses either Python or Go.

## Requirements

- LLM Agent with write permissions
- Already authenticated with **okta.py**

## Golden Image Reference

### Python Golden Images
- **ECR Repository**: `482234811004.dkr.ecr.us-east-1.amazonaws.com/golden-image-python`
- **Available Tags**:
  - `stable-debian`, `stable-alpine` (for production)
  - `rc-debian`, `rc-alpine` (for testing)
  - Version-specific: `3.14.0-debian`, `3.14.0-alpine`, etc.
- **AVOID `3.13.7-debian`** — this version is broken

### Go Golden Images
- **ECR Repository**: `482234811004.dkr.ecr.us-east-1.amazonaws.com/golden-image-go`
- **Available Tags**: Similar pattern with Go version numbers (e.g., `1.24.1-debian`, `stable-debian`)

### Key Details
- All images have a `/app` directory accessible to the non-root `shippo` user
- Default user is `shippo` (non-root, UID 10000)
- When installing packages, switch to `root` user, then back to `shippo`
- Use ARG statements at the top of Dockerfile for flexibility

---

## Plan File

All progress is tracked in a plan file. Create it at the start of Phase 1:

```
.claude/plans/<ticket-id>/<ticket-id>-golden-image-migration.md
```

### Plan File Template

```markdown
# <ticket-id>: Golden Image Migration — <service-name>

## Status: Phase 1 — Discovery
<!-- Update this line as you progress through phases -->

## Service Info
- **Repository**: <repo-name>
- **Language**: <Python/Go>
- **Current version**: <X.Y.Z>
- **Target golden image**: <TBD until Phase 2>
- **Dockerfile location**: <path>

## Phase Tracking

### Phase 1: Discovery & Version Identification
- [ ] Identified language and current version
- [ ] Located Dockerfile
- [ ] Documented findings in this file
- [ ] User approved findings

### Phase 2: Golden Image Selection
- [ ] Authenticated with ECR
- [ ] Queried available versions
- [ ] Selected target version: <version>
- [ ] User approved target version

### Phase 3: Dockerfile Migration
- [ ] Updated FROM statement with ARG pattern
- [ ] Removed redundant setup steps
- [ ] Updated package installation with proper user switching
- [ ] Verified /app directory usage and final USER is shippo
- [ ] User approved Dockerfile changes

### Phase 4: Dependency Updates
- [ ] Cleared Poetry/module cache
- [ ] Updated pyproject.toml / go.mod constraints
- [ ] Regenerated lock file
- [ ] Verified resolved versions (shippo-core, grpcio if Python)
- [ ] Dependencies install successfully
- [ ] User approved dependency changes

### Phase 5: Breaking Changes Review
- [ ] Identified version upgrade range (X.Y → A.B)
- [ ] Searched for breaking changes via web search
- [ ] Searched codebase for deprecated patterns
- [ ] Fixed or documented all identified issues
- [ ] User confirmed breaking changes addressed

### Phase 6: CI/CD Workflow Updates
- [ ] Located workflow files
- [ ] Added ECR login step to each workflow
- [ ] Added permissions (id-token: write, contents: read)
- [ ] Updated build step with ECR_REPO/TAG build args
- [ ] Noted if OIDC role update needed
- [ ] User approved workflow changes

### Phase 7: Verification & Validation
- [ ] `make check-ci` passes
- [ ] Docker image builds locally
- [ ] All tests pass
- [ ] User approved final changes

### Phase 8: PR Creation
- [ ] Created draft PR
- [ ] PR description references ticket and summarizes changes
- [ ] Shared PR link with user

## Deviations & Notes
<!-- Record anything that deviated from the standard process -->

## Files Modified
<!-- Updated as changes are made -->
```

**Update the plan file after every completed checklist item.** This is the source of truth for resuming work.

---

## Execution Rules

### Human-in-the-Loop Gates

Each phase ends with a **STOP** gate. You MUST:
1. Update the plan file with completed items
2. Present a brief summary (under 200 words) of what was done
3. **Wait for explicit user approval** before starting the next phase

Do NOT proceed to the next phase without user confirmation.

### Context Window Management

This migration can be lengthy. After completing each phase:

1. If you estimate context usage is high, tell the user:
   > "Context window is getting full. I recommend starting a fresh session. The plan file at `.claude/plans/<ticket-id>/<ticket-id>-golden-image-migration.md` has the current status. Use this prompt to resume:
   >
   > `Continue the golden image migration for <ticket-id>. The plan file is at .claude/plans/<ticket-id>/<ticket-id>-golden-image-migration.md — read it and pick up from the current phase.`"

2. If resuming from a plan file, read it first, identify the current phase from the `## Status` line and unchecked items, and continue from there.

### Response Length
- Keep phase summaries under 200 words
- Bullet points over paragraphs

---

## Phase 1: Discovery & Version Identification

1. Determine language: check for `pyproject.toml` or `go.mod`, check Dockerfile for base image
2. Extract current version:
   - **Python**: `FROM python:X.Y.Z-slim-bookworm` in Dockerfile, `python = ">=X.Y.Z,<4.0"` in pyproject.toml
   - **Go**: `FROM golang:X.Y.Z` in Dockerfile, `go X.Y` in go.mod
3. Locate Dockerfile (typically `config/Dockerfile` or root `Dockerfile`)
4. **Create the plan file** with findings filled in

**STOP** — Present findings and wait for user confirmation.

---

## Phase 2: Golden Image Selection

1. Authenticate with ECR (if needed):
   ```bash
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 482234811004.dkr.ecr.us-east-1.amazonaws.com
   ```

2. Query available versions:
   - **Python:**
     ```bash
     aws ecr describe-images --repository-name golden-image-python --region us-east-1 \
       --query 'sort_by(imageDetails,&imagePushedAt)[*].imageTags[0]' --output text \
       | grep -E '^[0-9]+\.[0-9]+\.[0-9]+-debian$' | sort -V | tail -10
     ```
   - **Go:**
     ```bash
     aws ecr describe-images --repository-name golden-image-go --region us-east-1 \
       --query 'sort_by(imageDetails,&imagePushedAt)[*].imageTags[0]' --output text \
       | grep -E '^[0-9]+\.[0-9]+\.[0-9]+-debian$' | sort -V | tail -10
     ```

3. Select the closest version >= current version
   - **Skip `3.13.7-debian`** (broken)
   - Prefer version-specific tags over `stable-*` for reproducibility

4. Update the plan file with selected target version

**STOP** — Present version selection rationale. Wait for user approval of the target version.

---

## Phase 3: Dockerfile Migration

1. Replace `FROM` statement with ARG pattern:
   ```dockerfile
   ARG ECR_REPO=482234811004.dkr.ecr.us-east-1.amazonaws.com/golden-image-python
   ARG TAG=<selected-version>
   FROM $ECR_REPO:$TAG AS base
   ```

2. Remove redundant setup:
   - Remove Python/Go installation steps (already in golden image)
   - Remove user creation (`adduser shippo`) — already exists
   - Keep only service-specific setup

3. Update package installation with proper user switching:
   ```dockerfile
   USER root
   RUN apt-get update && apt-get install -y <packages> && rm -rf /var/lib/apt/lists/*
   USER shippo
   ```
   - For Alpine: `RUN apk add --no-cache <packages>`

4. Verify `WORKDIR /app` is set and `USER shippo` is set before CMD/ENTRYPOINT

5. Update the plan file

**STOP** — Present the Dockerfile diff. Wait for user approval.

---

## Phase 4: Dependency Updates

### Python Services

1. Clear Poetry cache:
   ```bash
   poetry cache clear shippo --all
   poetry cache clear pypi --all
   ```

2. Update `pyproject.toml`:
   ```toml
   python = ">=<target-version>,<4.0"
   shippo-core = { version = "^1.0", source = "shippo", extras = ["http", "proto"] }
   shippo-kafka = { version = "^1.0", source = "shippo" }
   httpx = ">=0.26.0"
   greenlet = "^3.0"
   ```

3. Regenerate lock:
   ```bash
   poetry env use python<target-major.minor>
   rm poetry.lock
   poetry lock --no-cache
   poetry install
   ```

4. Verify resolved versions:
   ```bash
   grep -A2 'name = "shippo-core"\|name = "grpcio"' poetry.lock
   ```
   - `shippo-core` should be latest
   - `grpcio` should be >= 1.68.0

5. If old versions persist, force with exact pin as fallback

6. Check for PyO3 compatibility issues:
   ```bash
   poetry show --outdated | grep -E "(pydantic|pyo3|shippo)"
   ```
   - If dependencies don't support target Python, add to Dockerfile:
     ```dockerfile
     ENV PYO3_USE_ABI3_FORWARD_COMPATIBILITY=1
     ```
   - Document which packages required this flag

### Go Services
- Update Shippo Go modules to latest compatible versions
- Run: `go mod tidy && go mod download`

### Iterate until dependencies resolve successfully.

7. Update the plan file

**STOP** — Present dependency resolution results. Wait for user approval.

---

## Phase 5: Breaking Changes Review

**This phase is mandatory. Do not skip.**

1. Document the version upgrade range (e.g., "Python 3.11 -> 3.14")

2. Web search for breaking changes between the two versions. Review:
   - Official changelog / "What's New" docs
   - Deprecated features removed
   - Standard library changes
   - Behavioral changes, syntax changes, removed modules

3. Search codebase for deprecated patterns found in step 2. Examples:
   ```bash
   grep -r "utcnow" --include="*.py" .
   grep -r "asyncio.iscoroutinefunction" --include="*.py" .
   ```

4. Fix all identified issues

5. Update the plan file, including issues found and fixes applied

**STOP** — Present the breaking changes analysis and fixes. Wait for user confirmation that all issues are addressed.

---

## Phase 6: CI/CD Workflow Updates

1. Locate workflow files:
   - `.github/workflows/main.yaml`
   - `.github/workflows/dev-main.yaml`
   - `.github/workflows/check.yaml`

2. Add ECR login step BEFORE the build step in each:
   ```yaml
   - name: AWS Auth
     uses: 'goshippo/shippo-action-workflows/.github/actions/aws_auth@stable'
     id: aws-auth
     with:
       oidc-name: ${{ env.AWS_ROLE_NAME }}
       environment: dev-main
       login-to-ecr: true
   ```

3. Add permissions:
   ```yaml
   permissions:
     id-token: write
     contents: read
   ```

4. Update build step with build args if needed:
   ```yaml
   docker build \
     --build-arg ECR_REPO=${{ env.ECR_REPO }} \
     --build-arg TAG=${{ env.TAG }} \
     -f config/Dockerfile .
   ```

5. Check if OIDC role needs ECR pull permissions (note for potential `shippo-tf-services` PR)
   - Reference: https://github.com/goshippo/shippo-tf-services/pull/5341/changes

6. Update the plan file

**STOP** — Present workflow changes. Wait for user approval.

---

## Phase 7: Verification & Validation

1. Run checks iteratively:
   ```bash
   make check-ci
   ```

2. Fix any failures:
   - Linting: `make lint`
   - Types: `make mypy`
   - Tests: `make test`
   - Docker: `make docker-build`

3. Iterate until all checks pass

4. Verify Docker build locally (authenticate with ECR first if session expired)

5. Update the plan file with verification results

**STOP** — Present verification results. Wait for user approval to proceed to PR.

---

## Phase 8: PR Creation

1. Create a draft pull request against the default branch
2. PR description should:
   - Reference the Jira ticket
   - Summarize the migration (version change, key dependency updates, breaking changes fixed)
   - List all files modified
3. Update the plan file with the PR link

**STOP** — Share PR link with user.

---

## Expected File Changes

Files typically modified:
1. `config/Dockerfile` (or `Dockerfile`) — base image update
2. `pyproject.toml` (Python) or `go.mod` (Go) — dependency updates
3. `poetry.lock` (Python) or `go.sum` (Go) — lock file updates
4. `.github/workflows/main.yaml` — ECR login addition
5. `.github/workflows/dev-main.yaml` — ECR login addition
6. Potentially source code files if breaking changes require fixes

## Important Notes

- **shippo-core version is critical**: Clear Poetry cache before locking. Older versions pin `grpcio` 1.54.x which doesn't support Python 3.14. Latest has `grpcio >= 1.78.0`.
- **User switching**: Always switch back to `shippo` user after root operations
- **App directory**: Use `/app` for application code (already exists in golden image)
- **Local vs CI**: ARG defaults work for local builds; CI should override with environment-specific values
- **Testing**: After PR merges, deploy to dev-main for real-world validation
- **RC vs Stable**: Use `rc-*` tags for testing, `stable-*` for production
- **Local Python**: Set local Python to target version before running `poetry lock`

## References

- [Golden Images Guide](https://shippo.atlassian.net/wiki/spaces/SRE/pages/4260856537/Build+With+Golden+Images)
- [Migration Guide](https://shippo.atlassian.net/wiki/spaces/CET/pages/4609015848/Migrating+Microservices+to+Golden+Images+A+Step-by-Step+Guide)
- [Golden Image Source](https://github.com/goshippo/sre-docker/tree/main/golden)
- [Build Action](https://github.com/goshippo/shippo-action-workflows/tree/main/.github/actions/build_and_push_docker)
- [OIDC Role PR Reference](https://github.com/goshippo/shippo-tf-services/pull/5341/changes)
