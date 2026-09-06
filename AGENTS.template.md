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

<!-- @agentsnippet "@/pull-request-delivery-workflow.md" -->
