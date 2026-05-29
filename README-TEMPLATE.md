# Android SDK App Template

This repository is the **CloudOps Works Android SDK application template** for bootstrapping a new Android application with a Gradle/Kotlin DSL project, GitHub Actions workflows, and CloudOps Works CI/CD delivery wiring already in place.

Use this template when you want a repository that already includes:

- an Android Gradle build pipeline (Kotlin DSL) producing APK/AAB artifacts
- CloudOps Works CI/CD configuration under `.cloudopsworks/`
- GitHub Actions workflows for PR validation, main builds, device testing, and cleanup
- deployment templates for AWS Device Farm and GCP Firebase Test Lab
- Dual GitVersion presets (GitFlow and GitHub Flow)
- optional security scanning via Snyk, Semgrep, and SonarQube

---

## What gets generated from this template

### Application scaffold
- `app/` — Android application module (Kotlin)
- `buildscripts/` — shared Gradle build logic and convention plugins
- `build.gradle.kts` — root Gradle build file
- `settings.gradle.kts` — module declarations and plugin management
- `gradle.properties` — Gradle JVM and build flags
- `gradlew` / `gradlew.bat` — Gradle wrapper scripts
- `spotless/` — code style and formatting configuration
- `Makefile` — version helper targets powered by Tronador

### Delivery scaffold
- `.cloudopsworks/cloudopsworks-ci.yaml` — repository governance and deployment routing
- `.cloudopsworks/vars/inputs-global.yaml` — global Android SDK config, cloud target, security toggles
- `.cloudopsworks/vars/inputs-ANDROID-ENV.yaml` — per-environment Android build and device configuration
- `.cloudopsworks/vars/preview/` — preview environment configuration fragments
- `.cloudopsworks/_VERSION` — template version tracked by release automation
- `.cloudopsworks/gitversion_gitflow.yaml` — GitFlow reference configuration
- `.cloudopsworks/gitversion_githubflow.yaml` — GitHub Flow reference configuration
- `.github/workflows/` — reusable CI/CD orchestration

---

## Recommended bootstrap flow

### 1. Create a repository from this template
Create a new repository from `cloudopsworks/androidsdk-app-template`, then clone it locally.

### 2. Initialize the project metadata
```bash
make code/init
```
This sets the correct project name and version derived from GitVersion.

### 3. Replace the sample Android application
The template includes a JetSnack sample app so the build pipeline works immediately. Replace it with your actual Android project:

- update `app/build.gradle.kts` — set `applicationId`, `compileSdk`, `targetSdk`, `minSdk`, signing config
- update `settings.gradle.kts` — set `rootProject.name` and list real modules
- update `gradle.properties` — set project-specific flags
- replace the `app/` source code with your real application code

### 4. Choose a branching model
Two GitVersion presets are included. Copy the one matching your strategy:

```bash
# For GitFlow (develop, feature/*, release/*, hotfix/*)
cp .cloudopsworks/gitversion_gitflow.yaml .cloudopsworks/gitversion.yaml

# For GitHub Flow (main/master, short-lived feature branches)
cp .cloudopsworks/gitversion_githubflow.yaml .cloudopsworks/gitversion.yaml
```

### 5. Configure CloudOps Works settings
Update `.cloudopsworks/cloudopsworks-ci.yaml`:
- `branchProtection` — enable/disable branch protection
- `gitFlow.enabled` — set to `true` for GitFlow
- `requiredReviewers`, `reviewers`, `owners`, `contributors` — governance
- `cd.deployments` — branch/tag to environment mapping

### 6. Configure environment inputs
Set these values in `.cloudopsworks/vars/inputs-global.yaml` before the first pipeline run:
- `organization_name`
- `organization_unit`
- `environment_name`
- `repository_owner`
- `cloud` — `aws` or `gcp`
- `cloud_type` — `device-farm` (AWS) or `firebase-test-lab` (GCP)
- `android.version` — Android SDK command-line tools version number
- `android.sdks` — list of Android API levels to install
- `android.destinations` — emulator/device targets for local runs

### 7. Configure per-environment inputs
Fill in `.cloudopsworks/vars/inputs-ANDROID-ENV.yaml` for each environment:
- `environment` — target environment label
- `android.configuration` — `Debug` or `Release`
- For AWS Device Farm: uncomment and fill the `aws` block
- For GCP Firebase Test Lab: uncomment and fill the `gcp` block

---

## Deployment targets

### AWS Device Farm
Use the `aws` block in `inputs-ANDROID-ENV.yaml`.

Key fields:
- `aws.region` — must be `us-west-2` (Device Farm requirement)
- `aws.device_farm_name` — name of the Device Farm project
- `aws.device_farm_pool` — device pool (or specify `aws.device` individually)
- `aws.test_script_path` — path to test artifacts
- `aws.test_script_type` — test framework type (e.g., `INSTRUMENTATION_TEST_PACKAGE`)

To discover available devices:
```bash
aws devicefarm list-devices --region us-west-2
```

### GCP Firebase Test Lab
Use the `gcp` block in `inputs-ANDROID-ENV.yaml`.

Key fields:
- `gcp.project_id` — GCP project ID hosting Firebase Test Lab
- `gcp.region` — GCP region
- `gcp.devices` — list of `model_id`, `version_id`, `locale_id`, `orientation` entries
- `gcp.test_lab_bucket` — GCS bucket for test results
- `gcp.test_script_type` — test type: `robo`, `instrumentation`, or `game-loop`

To discover available Android devices:
```bash
gcloud firebase test android models list
```

---

## GitHub Actions workflow model

Key workflows in this template:

- `main-build.yml` — full Gradle build, artifact upload, GitVersion tagging on branch/tag events
- `pr-build.yml` — PR validation: lint, compile, unit test, security scan
- `deploy.yml` — dispatched per environment; runs the test suite against real devices
- `scan.yml` — periodic security and dependency scanning
- `automerge.yml` — auto-merge approved dependency-update PRs
- `pr-close.yaml` — preview environment cleanup when a PR closes
- `jira-integration.yml` — Jira ticket transitions on release events
- `environment-destroy.yml` / `environment-unlock.yml` — environment lifecycle operations

---

## Minimum checklist before first release

- [ ] Replace the JetSnack sample with the real Android application source
- [ ] Set `applicationId`, `compileSdk`, `targetSdk`, and signing config in `app/build.gradle.kts`
- [ ] Choose a GitVersion preset and copy it to `.cloudopsworks/gitversion.yaml`
- [ ] Replace all placeholders in `inputs-global.yaml` (`ORG_NAME`, `ORG_UNIT`, `ENV_NAME`, `REPO_OWNER`)
- [ ] Set real `cloud` and `cloud_type` values in `inputs-global.yaml`
- [ ] Fill in `inputs-ANDROID-ENV.yaml` with real environment, device, and cloud values
- [ ] Configure GitHub environment secrets for each deployment environment
- [ ] Update `README.md` to describe the actual application
- [ ] CI passes on a pull request to main/master

---

## Notes

- `.omx/`, `.claude/`, `.opencode/`, and similar agent/tooling directories are intentionally ignored and are not part of the application template contract.
- The template is designed for CloudOps Works blueprint-backed automation; if you remove that integration, also prune the related workflows and `.cloudopsworks/` configuration.

---

## AI-assisted upgrade of `.cloudopsworks/vars` configuration files

This section is a machine-readable protocol for AI agents performing a seamless, non-destructive upgrade of all configuration files under `.cloudopsworks/vars/` when a new template version is released. Follow the steps below in order.

### Upgrade overview

The template version locked into this repository is recorded in `.cloudopsworks/_VERSION`. The canonical upstream source is the GitHub repository `cloudopsworks/androidsdk-app-template`, pinned to the tag that matches the content of `_VERSION`.

An upgrade merges new keys, updated comments, and structural changes from the upstream template into local files **without overwriting values the operator has already set**.

---

### Step 1 — determine current and target versions

1. Read `.cloudopsworks/_VERSION` to get the **current locked version** (e.g., `v1.4.15`).
2. The **target version** is either supplied by the operator or is the latest release tag on `cloudopsworks/androidsdk-app-template`.
3. Fetch any upstream file from GitHub using the pattern:
   ```
   https://raw.githubusercontent.com/cloudopsworks/androidsdk-app-template/<version>/<path>
   ```
   Example:
   ```
   https://raw.githubusercontent.com/cloudopsworks/androidsdk-app-template/v1.4.15/.cloudopsworks/vars/inputs-global.yaml
   ```

---

### Step 2 — identify the deployment type for each environment file

Each `inputs-<name>.yaml` file under `.cloudopsworks/vars/` maps to a specific upstream template. Determine the type using the following priority order:

**Priority 1 — `Agents:` header comment**

If the file contains an `# Agents:` line in its header block, read `cloud` and `cloud_type` directly from it:

```yaml
# Agents: cloud=aws ; cloud_type=device-farm
```

**Priority 2 — fallback to `inputs-global.yaml`**

If no `# Agents:` line is present, read the active `cloud` and `cloud_type` values from `.cloudopsworks/vars/inputs-global.yaml` and apply the mapping table below.

**`cloud` / `cloud_type` → upstream template file:**

| `cloud`  | `cloud_type`                    | Upstream template file         |
|----------|---------------------------------|--------------------------------|
| `aws`    | `device-farm`                   | `inputs-ANDROID-ENV.yaml`      |
| `gcp`    | `firebase-test-lab`             | `inputs-ANDROID-ENV.yaml`      |

`inputs-global.yaml` always maps to the upstream `inputs-global.yaml` regardless of cloud type.

---

### Step 3 — upgrade deployment target files

The deployment target files identified by the Step 2 mapping table — such as `inputs-KUBERNETES-ENV.yaml`, `inputs-LAMBDA-ENV.yaml`, `inputs-BEANSTALK-ENV.yaml`, `inputs-APPENGINE.yaml`, `inputs-CLOUDRUN.yaml`, `inputs-LIB-ENV.yaml`, and mobile equivalents such as `inputs-ANDROID-ENV.yaml` and `inputs-XCODE-ENV.yaml` — are **scaffolding templates**. They provide placeholder structures and documented examples, not finalized operator configuration.

**Do not merge these files. Overwrite them.**

Upgrade procedure for each deployment target file:

1. **Before overwriting** — inspect the local file and record any operator-configured values (keys that have been uncommented and set to non-placeholder values).
2. **Replace the file** — overwrite the local file entirely with the upstream template version.
3. **Re-apply operator values** — after overwriting, set each previously recorded operator-configured value at its corresponding key in the new file.
4. **Copy in absent files** — if a deployment target file is present in the upstream template but absent locally, copy it in from the upstream template as a new file.

---

### Step 4 — merge `inputs-global.yaml`

`inputs-global.yaml` requires special handling because it contains mandatory operator identity fields alongside a large body of optional commented-out sections.

Merge procedure:

1. **Retain the four mandatory identity fields** verbatim at the top of the file:
   ```yaml
   organization_name: "..."
   organization_unit: "..."
   environment_name: "..."
   repository_owner: "..."
   ```
2. **Retain `cloud` and `cloud_type`** exactly as the operator set them.
3. **For every optional commented-out section** in the upstream template, check the local file:
   - If the operator **has uncommented and configured it** — keep the operator's values; update only surrounding comment text if it changed upstream.
   - If the section **is still fully commented out locally** — replace the entire commented block with the upstream version, capturing any new fields or updated documentation within it.
4. **Append new optional sections** that appear in the upstream template but are entirely absent locally, in fully commented-out form, preserving their upstream position and comments.

---

### Step 5 — upgrade subdirectory files

Apply the merge rules from Step 4 to every file in the following subdirectories, matching each local file to its corresponding upstream file at the same relative path:

- `.cloudopsworks/vars/preview/inputs.yaml`
- `.cloudopsworks/vars/preview/values.yaml`

---

### Step 6 — update `_VERSION`

After all merges are verified correct, write the target version string (e.g., `v1.4.16`) to `.cloudopsworks/_VERSION`. This is the final step.

---

### Upgrade invariants

An agent performing this upgrade must **never**:

- Overwrite a field the operator has explicitly set to a non-placeholder value.
- Remove a commented-out operator value without first reporting it.
- Change the YAML structure of any active (uncommented) operator section.
- Alter a file's opening description comment (`# This file contains...`) unless the upstream version changed it.
- Modify `.cloudopsworks/cloudopsworks-ci.yaml`, `gitversion_*.yaml`, or any file under `.github/workflows/` as part of a vars upgrade — those follow their own upgrade path.
- Update `_VERSION` before all file merges are complete.

---

### Conflict resolution

When a merge cannot be resolved automatically (for example, the upstream template restructured a section that the operator has customized):

1. Emit a diff showing both the upstream template block and the local operator block side by side.
2. Pause and present the conflict to the operator, asking which version to keep or whether a manual merge is needed.
3. Never silently choose one side.

---

## Release Workflow — use `cw-release`

All releases **must** be performed using the `cw-release` skill from the CloudOps Works skill set. Do **not** create release branches, hotfix branches, version tags, or release PRs manually — the skill owns the full GitFlow-aware release lifecycle for this repository.

### When to invoke `cw-release`

Use it whenever you are asked to:
- Release, ship, or publish a new version (patch, minor, or major)
- Create a hotfix or patch release
- Create a release branch or feature-merge PR
- Tag and publish a version

### How to run it

In Claude Code (CLI, IDE extension, or web):

```
/cw-release
```

### What the skill does

1. Detects the GitVersion flow in use (`gitversion_gitflow.yaml` or `gitversion_githubflow.yaml`).
2. Reads the repo-local release policy from `.cloudopsworks/cloudopsworks-ci.yaml`.
3. Drives the shared tronador `make` / `gh` release path end-to-end.
4. Creates the correct branch, PR, tag, and GitHub Release in the right sequence.

> **Do not** run `git tag`, `gh release create`, or `make release` directly. Always let `cw-release` orchestrate these steps to keep version history and CI consistent.
