---
name: migrate-golden-image
description: Migrate a Shippo microservice from custom Docker base image to standardized golden image
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git *, make *, aws ecr *, docker *, poetry *, go mod *, grep *), Task, WebSearch
---

# Migrate Microservice to Golden Images - Complete Migration Prompt

## Context

You are migrating a Shippo microservice from a custom Docker base image to the standardized golden image. This microservice follows the py-zoo template structure (https://github.com/goshippo/py-zoo) and uses either Python or Go.

## Golden Image Information

### Python Golden Images
- **ECR Repository**: `482234811004.dkr.ecr.us-east-1.amazonaws.com/golden-image-python`
- **Available Tags**:
  - `stable-debian`, `stable-alpine` (for production)
  - `rc-debian`, `rc-alpine` (for testing)
  - Version-specific: `3.14.0-debian`, `3.14.0-alpine`, etc.
  - Legacy: `3.13.7-debian`, `3.13.7-alpine` (temporary use only)

### Go Golden Images
- **ECR Repository**: `482234811004.dkr.ecr.us-east-1.amazonaws.com/golden-image-go`
- **Available Tags**: Similar pattern with Go version numbers (e.g., `1.24.1-debian`, `stable-debian`)

### Key Golden Image Details
- All images have a `/app` directory accessible to the non-root `shippo` user
- Default user is `shippo` (non-root, UID 10000)
- When installing packages, switch to `root` user, then back to `shippo` user
- Use ARG statements at the top of Dockerfile for flexibility (default to dev account for local builds)
- **AVOID `3.13.7-debian`** - This version is broken and should not be used

## Migration Steps

### Step 1: Identify Current Language and Version

1. **Determine if Python or Go:**
   - Check for `pyproject.toml` or `go.mod` file
   - Check `config/Dockerfile` or `Dockerfile` for base image

2. **Extract Current Version:**
   - **Python**: Look for `FROM python:X.Y.Z-slim-bookworm` or similar in Dockerfile
   - **Go**: Look for `FROM golang:X.Y.Z` or similar in Dockerfile
   - Also check `pyproject.toml` for `python = ">=X.Y.Z,<4.0"` constraint
   - Also check `go.mod` for `go X.Y` version

3. **Document Findings:**
   - Current language: [Python/Go]
   - Current version: [X.Y.Z]
   - Dockerfile location: [path]

### Step 2: Find Matching Golden Image

1. **Authenticate with ECR (if not already authenticated):**
   ```bash
   # Login with Okta to get AWS credentials
   okta.py <profile-name>

   # Login to ECR
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 482234811004.dkr.ecr.us-east-1.amazonaws.com
   ```

2. **Query ECR for Available Golden Image Versions:**
   - **For Python:**
     ```bash
     aws ecr describe-images --repository-name golden-image-python --region us-east-1 --query 'sort_by(imageDetails,&imagePushedAt)[*].imageTags[0]' --output text | grep -E '^[0-9]+\.[0-9]+\.[0-9]+-debian$' | sort -V | tail -10
     ```
   - **For Go:**
     ```bash
     aws ecr describe-images --repository-name golden-image-go --region us-east-1 --query 'sort_by(imageDetails,&imagePushedAt)[*].imageTags[0]' --output text | grep -E '^[0-9]+\.[0-9]+\.[0-9]+-debian$' | sort -V | tail -10
     ```
   - This will show the latest version-specific tags (e.g., `3.14.2-debian`, `1.24.1-debian`)

3. **Find Closest Matching Version:**
   - Compare your current version (from Step 1) with available versions from ECR
   - Select the closest version that is >= your current version
   - **IMPORTANT**: Skip `3.13.7-debian` - this version is broken and should not be used
   - Example: If current is Python 3.11.11 and available versions are `3.13.7-debian`, `3.14.0-debian`, `3.14.2-debian`, choose `3.14.0-debian` (skip 3.13.7)
   - Prefer version-specific tags (e.g., `3.14.0-debian`) over `stable-debian` for better reproducibility
   - If no exact match, choose the next minor or major version that's available (excluding broken versions)

4. **Select Golden Image Tag:**
   - **Recommended**: Use version-specific tag (e.g., `3.14.2-debian`) for closest match to current version
   - **Alternative**: Use `stable-debian` or `stable-alpine` if you want to always use latest stable
   - **Testing**: Use `rc-debian` or `rc-alpine` only if specifically testing release candidates

### Step 3: Update Dockerfile

1. **Locate Dockerfile:**
   - Typically at `config/Dockerfile` or root `Dockerfile`

2. **Update Base Image Section:**
   - Replace the `FROM` statement with ARG pattern using the closest matching version found in Step 2:

   **For Python (example with closest version):**
   ```dockerfile
   ARG ECR_REPO=482234811004.dkr.ecr.us-east-1.amazonaws.com/golden-image-python
   ARG TAG=3.14.2-debian
   FROM $ECR_REPO:$TAG AS base
   ```
   - Replace `3.14.2-debian` with the actual closest version you discovered from ECR

   **For Go (example with closest version):**
   ```dockerfile
   ARG ECR_REPO=482234811004.dkr.ecr.us-east-1.amazonaws.com/golden-image-go
   ARG TAG=1.24.1-debian
   FROM $ECR_REPO:$TAG AS base
   ```
   - Replace `1.24.1-debian` with the actual closest version you discovered from ECR

3. **Remove Redundant Setup:**
   - Remove Python/Go installation steps (already in golden image)
   - Remove user creation (`adduser shippo`) - already exists in golden image
   - Keep only service-specific setup (package installations, etc.)

4. **Update Package Installation (if needed):**
   - If installing system packages, ensure proper user switching:
   ```dockerfile
   USER root
   RUN apt-get update && apt-get install -y <packages> && rm -rf /var/lib/apt/lists/*
   USER shippo
   ```
   - For Alpine: `RUN apk add --no-cache <packages>`

5. **Ensure /app Directory Usage:**
   - Verify `WORKDIR /app` is set
   - Ensure all COPY commands target `/app` or subdirectories

6. **Verify Final User:**
   - Ensure `USER shippo` is set before CMD/ENTRYPOINT

### Step 4: Update Dependencies

1. **For Python Services (pyproject.toml):**

   #### Critical: Ensure Latest shippo-core Version

   **Warning: This is the most important step for Python 3.14 compatibility.**

   Older versions of `shippo-core` pin old `grpcio`/`grpcio-tools` (1.54.x) which do NOT have Python 3.14 wheels and will fail to compile. The latest `shippo-core` has updated these constraints (grpcio >= 1.78.0).

   a. **Clear Poetry cache first (prevents stale version resolution):**
      ```bash
      poetry cache clear shippo --all
      poetry cache clear pypi --all
      ```

   b. **Update pyproject.toml with these constraints:**
      ```toml
      python = ">=3.14,<4.0"  # Match golden image Python version
      shippo-core = { version = "^1.0", source = "shippo", extras = ["http", "proto"] }
      shippo-kafka = { version = "^1.0", source = "shippo" }
      httpx = ">=0.26.0"  # Relax to allow latest shippo-core's httpx requirement (0.28.1)
      greenlet = "^3.0"   # Update for Python 3.14 compatibility (2.x doesn't work)
      ```

   c. **Set local Python to 3.14 and regenerate lock:**
      ```bash
      poetry env use python3.14
      rm poetry.lock
      poetry lock --no-cache
      poetry install
      ```

   d. **Verify the resolved versions:**
      ```bash
      grep -A2 'name = "shippo-core"\|name = "grpcio"' poetry.lock
      ```
      - `shippo-core` should be the latest (e.g., `1.0.20260203184605`)
      - `grpcio` should be >= 1.68.0 (has Python 3.14 wheels)
      - `grpcio-tools` should be >= 1.68.0

   e. **If still getting old version, force with exact pin (fallback):**
      ```bash
      # Find latest version
      poetry add "shippo-core[http,proto]@latest" --source shippo --dry-run

      # Then pin to that exact version in pyproject.toml if needed
      shippo-core = { version = "1.0.YYYYMMDDHHMMSS", source = "shippo", extras = ["http", "proto"] }
      ```

2. **For Go Services (go.mod):**
   - Update any Shippo Go modules to latest compatible versions
   - Run: `go mod tidy && go mod download`

3. **Iterate Until Dependencies Resolve (Python):**
   - Run dependency update command
   - Check for errors
   - If errors occur, investigate and fix:
     - Version conflicts
     - Incompatible dependencies
     - Missing dependencies
   - Repeat until `make install` (or equivalent) succeeds

### Step 5: Check and Upgrade Dependencies for Python Compatibility

1. **Check Current Dependency Versions:**
   ```bash
   poetry show --outdated | grep -E "(pydantic|pyo3|shippo)"
   ```

2. **Attempt Dependency Upgrades:**
   - Update all dependencies to their latest versions:
     ```bash
     poetry update
     ```
   - Check if newer versions of critical packages support the target Python version
   - Key packages to check for Python 3.14 compatibility:
     - `pydantic` / `pydantic-core` (uses PyO3 Rust bindings)
     - `cryptography` (uses Rust)
     - Any package with native C/Rust extensions

3. **If Dependencies Don't Support Target Python Version:**
   - **Option A (Preferred)**: Wait for upstream support or use a compatible Python version
   - **Option B (Workaround)**: Set `PYO3_USE_ABI3_FORWARD_COMPATIBILITY=1` environment variable in Dockerfile:
     ```dockerfile
     # Only use this if dependencies cannot be upgraded to support Python 3.14
     ENV PYO3_USE_ABI3_FORWARD_COMPATIBILITY=1
     ```
   - This workaround allows PyO3-based packages to build with unsupported Python versions using the stable ABI

4. **Document Dependency Status:**
   - Note which packages required the compatibility flag
   - Track upstream issues for Python version support

### Step 5b: Review Codebase for Breaking Changes

**MANDATORY STEP - DO NOT SKIP**

> **This step is REQUIRED and must be completed before proceeding. Skipping this step can cause runtime failures in production that are not caught by tests.**

1. **Identify Version Upgrade:**
   - Document: "Upgrading from Python X.Y.Z to Python A.B.C" (or Go equivalent)
   - Example: "Upgrading from Python 3.11.11 to Python 3.14.2"

2. **Use Web Search to Review Breaking Changes:**
   - Search for "Python X.Y to A.B breaking changes" (e.g., "Python 3.11 to 3.14 breaking changes")
   - Review official Python/Go changelog and "What's New" documentation
   - Check for:
     - Deprecated features being used (e.g., `datetime.utcnow()` removed in 3.12+)
     - Standard library changes (asyncio, typing, dataclasses for Python)
     - Behavioral changes in existing functionality
     - Syntax changes
     - Removed modules or functions

3. **Search Codebase for Potential Issues:**
   - Search for deprecated features identified in step 2
   - Check type hints compatibility
   - Review async/await patterns (Python)
   - Check error handling patterns
   - Example searches:
     ```bash
     grep -r "utcnow" --include="*.py" .
     grep -r "asyncio.iscoroutinefunction" --include="*.py" .
     ```

4. **Fix Identified Issues:**
   - Update deprecated code
   - Fix type compatibility issues
   - Update syntax if needed
   - Document all changes made

5. **Confirm Completion:**
   - [ ] Reviewed official changelog for breaking changes
   - [ ] Searched codebase for deprecated patterns
   - [ ] Fixed or documented all identified issues
   - **You must complete this checklist before proceeding to Step 6**

### Step 6: Update CI/CD Workflows

1. **Locate Workflow Files:**
   - `.github/workflows/main.yaml` (or `main.yml`)
   - `.github/workflows/dev-main.yaml` (or `dev-main.yml`)
   - `.github/workflows/check.yaml` (or `check.yml`)

2. **Add ECR Login Step:**
   - Find the build job in each workflow
   - Add AWS authentication step BEFORE the build step:

   ```yaml
   - name: AWS Auth
     uses: 'goshippo/shippo-action-workflows/.github/actions/aws_auth@stable'
     id: aws-auth
     with:
       oidc-name: ${{ env.AWS_ROLE_NAME }}
       environment: dev-main  # or appropriate environment
       login-to-ecr: true
   ```

   - Ensure `permissions:` section includes:
     ```yaml
     permissions:
       id-token: write
       contents: read
     ```

3. **Update Build Step (if needed):**
   - Add build args for ECR_REPO and TAG if building in CI:
   ```yaml
   - name: Build docker image
     run: |
       docker build \
         --build-arg ECR_REPO=${{ env.ECR_REPO }} \
         --build-arg TAG=${{ env.TAG }} \
         -f config/Dockerfile .
   ```

4. **Verify OIDC Role Permissions:**
   - Check if OIDC role has ECR pull permissions for golden images
   - File location: `services/github-oidc-roles/vars.tf` in shippo-tf-services repo
   - If missing, note that a PR to shippo-tf-services may be needed
   - Reference: https://github.com/goshippo/shippo-tf-services/pull/5341/changes

### Step 7: Run Makefile Checks Iteratively

1. **Run Initial Check:**
   ```bash
   make check-ci
   ```
   Or if that doesn't exist:
   ```bash
   make check
   ```

2. **If Errors Occur, Fix and Retry:**
   - **Linting Errors**: Run `make lint` and fix issues
   - **Type Errors**: Run `make mypy` (Python) or equivalent and fix
   - **Test Failures**: Run `make test` and fix failing tests
   - **Docker Build Errors**: Run `make docker-build` and fix Dockerfile issues
   - **Dependency Errors**: Re-run `make update-deps` and resolve conflicts

3. **Iterate Until All Checks Pass:**
   - Run `make check-ci` after each fix
   - Continue until exit code is 0
   - Document any fixes made

4. **Verify Docker Build Locally:**
   - **Important**: Before building locally, you must authenticate with ECR to pull the golden image:
     ```bash
     # Login with Okta to get AWS credentials
     okta.py <profile-name>

     # Login to ECR
     aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 482234811004.dkr.ecr.us-east-1.amazonaws.com
     ```
   - Then build the Docker image:
     ```bash
     make docker-build
     ```
     Or:
     ```bash
     docker build -f config/Dockerfile .
     ```

### Step 8: Final Validation

1. **Verify All Changes:**
   - [ ] Dockerfile uses golden image with ARG pattern
   - [ ] Dependencies updated to latest available versions (for shippo-* packages)
   - [ ] `make check-ci` passes without errors
   - [ ] Docker build succeeds locally
   - [ ] CI workflows have ECR login step
   - [ ] No breaking changes introduced

2. **Create Summary:**
   - Document version upgrade: X.Y.Z -> A.B.C
   - List all files modified
   - Note any special considerations or fixes applied

## Expected File Changes

### Files Typically Modified:
1. `config/Dockerfile` (or `Dockerfile`) - Base image update
2. `pyproject.toml` (Python) or `go.mod` (Go) - Dependency updates
3. `poetry.lock` (Python) or `go.sum` (Go) - Lock file updates
4. `.github/workflows/main.yaml` - ECR login addition
5. `.github/workflows/dev-main.yaml` - ECR login addition
6. Potentially source code files if breaking changes require fixes

## Important Notes

- **Step 5b is MANDATORY**: You MUST complete the breaking changes review before proceeding. Do not skip this step under any circumstances. Runtime failures from deprecated APIs may not be caught by tests.
- **shippo-core Version is Critical**: Clear Poetry cache (`poetry cache clear shippo --all`) before locking to get the latest `shippo-core`. Older versions have pinned `grpcio` 1.54.x which doesn't support Python 3.14. The latest version has `grpcio >= 1.78.0`. If cache clearing doesn't work, pin to exact version as fallback.
- **User Switching**: Always switch back to `shippo` user after root operations
- **App Directory**: Use `/app` for application code (already exists in golden image)
- **Local vs CI**: ARG defaults work for local, CI should override with environment-specific values
- **Testing**: After PR merges, deploy to dev-main for real-world validation
- **RC vs Stable**: Use `rc-*` tags for testing, `stable-*` for production deployments
- **Local Python**: Set local Python to 3.14 (`poetry env use python3.14`) before running `poetry lock`

## Success Criteria

- Dockerfile successfully uses golden image
- All dependencies resolve and install correctly
- **Step 5b completed: Breaking changes reviewed and fixed** (MANDATORY)
- `make check-ci` passes completely
- Docker image builds successfully
- CI workflows include ECR authentication
- No breaking changes or deprecated features remain

## References

- Golden Images Guide: https://shippo.atlassian.net/wiki/spaces/SRE/pages/4260856537/Build+With+Golden+Images
- Migration Guide: https://shippo.atlassian.net/wiki/spaces/CET/pages/4609015848/Migrating+Microservices+to+Golden+Images+A+Step-by-Step+Guide
- Golden Image Source: https://github.com/goshippo/sre-docker/tree/main/golden
- Build Action: https://github.com/goshippo/shippo-action-workflows/tree/main/.github/actions/build_and_push_docker

---

**Now proceed with the migration following these steps in order. Work autonomously and report progress at each major step.**

**REMINDER: Step 5b (Review Codebase for Breaking Changes) is MANDATORY and must not be skipped.**
