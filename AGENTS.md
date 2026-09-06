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

## Image authoring reference

When adding an image or changing registry/version parsing, image README structure, or build wrappers, read [docs/image-authoring.md](docs/image-authoring.md). It contains the required templates, registry patterns, and architecture defaults.

Instruction-only edits need generated-file and link checks; run `./scripts/test-update.sh` for image or update-workflow changes.

## Pull Request Delivery Workflow

Deliver repository changes through pull requests by default, regardless of
size. Do not make changes directly in the main checkout unless the user
explicitly approves an exception. Direct commits to `main` or the default
branch should be limited to explicit user-approved exceptions.

Work on a dedicated topic branch, using a separate worktree when required or
useful. Make the requested change, run relevant validation, and pass the review
gate below before committing or creating/updating a PR. Keep saved-plan
progress current and close the plan when its objective is complete. PRs should
describe the final scope and validation results.

When asked to prepare changes as PRs for review, finish with validated,
reviewed PRs and report remaining limitations. A read-only review ends with
findings and coverage limits; it does not authorize changes or PR creation.
For authorized delivery, continue through green checks,
merge, any explicitly authorized deployment, and verified cleanup. Use a
Conventional Commit PR title and squash subject when the repository uses them
to determine release versions.

Treat a request to `deploy`, `ship`, `publish`, or `deliver` the current
requested repository change set as authorization to complete this normal
topic-branch workflow: commit reviewed in-scope changes, push the topic branch,
create or update its pull request, monitor required checks, make narrowly scoped
fixes for failures caused by the change, merge when all gates pass, and remove
the clean merged worktree and merged topic branches under the cleanup checks
below. Apply required validation and review to every fix. Do not ask for
separate approval for each ordinary step.

This authorization applies only to the current requested repository change
set. It does not authorize force pushes; bypassing reviews, checks, or branch
protections; direct-default-branch commits; manual releases or package
publication outside the repository's existing merge-triggered automation;
access to or disclosure of secrets; destructive repository operations;
unrelated pull requests; or material scope expansion. Authorization to merge a
pull request includes any package version and publication performed
automatically by the repository's existing merge workflow. In this section,
`deploy` authorizes repository delivery; it authorizes a service or
infrastructure deployment only when the current request specifically identifies
that deployment. More-specific repository approval rules, including final
content or product publication, still apply. Cleanup is limited to the verified merged worktree and topic
branches described below; it never includes dirty worktrees or forced remote operations.

When requesting platform approval for an authorized step, quote the user's
delivery request and this shared instruction in the justification. If a
platform reviewer rejects the action, ask the user once and wait. Do not retry
an equivalent escalation or repeat the prompt during automatic continuations
unless the user provides new authorization or relevant context.

Before committing, run `git status --short`, stage intended files by exact
path, and verify the staged scope. For an explicitly approved default-branch
exception, state that the normal PR workflow is being bypassed and still check
scope. Include screenshots only for changes to rendered UI, generated visual
output, or external presentation.

## Merged-Branch Cleanup

After confirming the exact PR is merged, remove only its clean worktree.
Ordinary remote branch deletion requires the remote ref to match the PR's
recorded head. A local topic branch may be deleted with `git branch -D` only
when its tip matches that recorded head and either:

- Its tree matches the squash commit's tree; or
- When the base advanced, both the `git patch-id --verbatim` of the aggregate
  diff from the merge base matches the squash commit's first-parent diff and
  applying that exact aggregate diff to the first-parent tree produces the
  squash commit's tree.

The second proof handles intervening base changes without ignoring whitespace
or patch locations. Retain the branch if neither proof succeeds. This is not
authorization for `git branch -D` on any other local branch or for other
destructive operations.

## Review Gate

Before committing, use the installed `$branch-review-subagent-loop` skill to
review the complete branch diff. Follow the skill through any required fixes,
validation, and re-review. If the skill is unavailable, ask the user to install
it before continuing.

Create, update, or merge the pull request only after the review gate passes.
Merging also requires green checks unless the user explicitly accepts the
remaining risk.
