# AGENTS.md

## Adding new images

When adding a new Docker image to this project:

- Create directory: `mkdir [image-name]`
- Create `[image-name]/Dockerfile`
- Add entry to `.github/config/images.yml`
- Create `.github/workflows/build-[image-name].yml` (simple wrapper that calls reusable workflow)
- Create `[image-name]/README.md` following existing pattern
- Update main `README.md` to include new image in "Images Included" section
- Test with `./scripts/test-update.sh`

## Registry support

- Docker Hub: Use image name only (e.g., `nginx`)
- GitHub Container Registry: Use full path (e.g., `ghcr.io/owner/repo/package`)

## Version parsing

- Docker Hub: `grep -oP '[image]:\K[0-9]+\.[0-9]+\.[0-9]+' $DOCKER_NAME/Dockerfile`
- GitHub Container Registry: `grep -oP 'ghcr\.io/[owner]/[repo]/[package]:\K[0-9]+\.[0-9]+\.[0-9]+' $DOCKER_NAME/Dockerfile`

## README template

Follow this exact structure for `[image-name]/README.md`:

```markdown
# [Image Name]

[![GitHub Container Registry](https://img.shields.io/badge/GHCR%20-%20favoyang%2Fcustom--images%2F[image-name]%20-%20%230db7ed?style=flat&logo=docker)](https://ghcr.io/favoyang/custom-images/[image-name])
[![GitHub build status](https://img.shields.io/github/actions/workflow/status/favoyang/custom-images/build-[image-name].yml?label=Build)](https://github.com/favoyang/custom-images/actions/workflows/build-[image-name].yml)

This image is updated automatically by GitHub Actions when changes are made to the Dockerfile using the official [Base Image](link-to-base) image.

## Usage

Docker builds are available at GitHub Container Registry:

- **GitHub Packages**: `docker pull ghcr.io/favoyang/custom-images/[image-name]:latest`

### Tags

The following tags are available for the `ghcr.io/favoyang/custom-images/[image-name]` image:

- `latest`
- `<version>` (eg: `1.0.0`, including: `1.0`, `1`, etc.)
```

## Workflow template

Create `.github/workflows/build-[image-name].yml` using this template:

```yaml
# Workflow to build and push [Image Name] Docker image to GitHub Container Registry
name: Build [image-name]

# Controls when the action will run
on:
  workflow_dispatch:  # allows to run the workflow manually from the Actions tab
  push:
    branches: main
    paths:
      - [image-name]/Dockerfile

# Permissions needed for this workflow
permissions:
  contents: read
  packages: write

jobs:
  build:
    uses: ./.github/workflows/build-image.yml
    with:
      docker_name: [image-name]
      docker_description: "[Description of the Docker image]"
      version_regex: '[appropriate-regex-pattern]'
      platforms: linux/amd64
```

## Key requirements

- Use the reusable workflow template above for new build workflows
- Choose appropriate `version_regex` pattern based on base image registry
- Default to `linux/amd64` architecture
- Test with `./scripts/test-update.sh` before committing

## Pull Request Delivery Workflow

Deliver repository changes through pull requests by default, regardless of
size. Do not make changes directly in the main checkout unless the user
explicitly approves an exception. Direct commits to `main` or the default
branch should be limited to explicit user-approved exceptions.

Follow this delivery sequence:

1. Create a dedicated topic branch. Use a separate worktree when repository
   guidance requires one or when isolation is useful.
2. Make the requested change and run relevant validation.
3. Update plan progress when working from a saved plan.
4. Run the review gate, fix valid findings, revalidate, and repeat the review
   until it passes.
5. Close the plan when appropriate, then commit and push the reviewed change.
6. Create or update the GitHub pull request with a brief summary and the
   validation commands that were run.
7. Verify required checks and merge when there is no blocking reason.
8. Monitor deployment when applicable, then clean up the worktree and branch.

Do not interpret short requests such as "commit", "publish the change", "ship
it", "push it", or "merge it" as approval to bypass this workflow. Unless the
user explicitly says to work directly on the default branch, skip the pull
request workflow, or make a direct-default-branch exception, continue the
normal flow on the dedicated topic branch, using a separate worktree when
applicable.

Direct-default-branch exceptions still need a clean scope check before
committing. When an exception is approved, state that the normal pull request
workflow is being bypassed because of the explicit exception.

Before committing, run `git status --short` and verify the staged files match
the requested change. Stage files by exact path when possible. Avoid broad
staging commands such as `git add .` when unrelated local work exists.

Include screenshots in the pull request only if a change affects rendered UI,
generated visual output, or external presentation.

## Review Gate

Before committing, use the installed `$branch-review-subagent-loop` skill to
review the complete branch diff. Follow the skill through any required fixes,
validation, and re-review. If the skill is unavailable, ask the user to install
it before continuing.

Create, update, or merge the pull request only after the review gate passes.
Merging also requires green checks unless the user explicitly accepts the
remaining risk.
