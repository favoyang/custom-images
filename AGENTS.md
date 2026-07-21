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
8. Monitor any explicitly authorized deployment when applicable, then remove
   the clean merged worktree and delete its merged local and remote topic
   branches. Ordinary remote deletion is authorized after confirming that the
   exact pull request is merged and the remote ref matches its recorded head.
   After a squash merge, `git branch -D` is authorized only for the local topic
   branch after confirming that its tip also matches the recorded head and its
   tree matches the merge commit's tree.

Treat a request to `deploy`, `ship`, `publish`, or `deliver` the current
requested repository change set as authorization to complete this normal
topic-branch workflow: commit reviewed in-scope changes, push the topic branch,
create or update its pull request, monitor required checks, make narrowly scoped
fixes for failures caused by the change, merge when all gates pass, and remove
the clean merged worktree and merged topic branches under the cleanup checks
above. Apply required validation and review to every fix. Do not ask for
separate approval for each ordinary step.

This authorization applies only to the current requested repository change
set. It does not authorize force pushes; bypassing reviews, checks, or branch
protections; direct-default-branch commits; releases or package publication;
access to or disclosure of secrets; destructive repository operations;
unrelated pull requests; or material scope expansion. Cleanup does not include
removing a dirty worktree, using `git branch -D` for any other local branch, any
forced remote operation, or other destructive operations. In this section,
`deploy` authorizes repository delivery; it authorizes a service or
infrastructure deployment only when the current request specifically identifies
that deployment. More-specific repository approval rules, including final
content or product publication, still apply.

When requesting platform approval for an authorized step, quote the user's
delivery request and this shared instruction in the justification. If a
platform reviewer rejects the action, ask the user once and wait. Do not retry
an equivalent escalation or repeat the prompt during automatic continuations
unless the user provides new authorization or relevant context.

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
