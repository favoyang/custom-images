# Custom Docker Images

## Personal Repository

This repository contains custom Docker images for personal use.

## Images Included

- **[caddy-cloudflare-geoip-ratelimit](./caddy-cloudflare-geoip-ratelimit)**: Custom Caddy server with Cloudflare, GeoIP, and rate limiting capabilities

## Automated Updates

Base images are automatically checked for updates every Sunday and updated when new versions are available. To include new images in automated updates, add them to `.github/config/images.yml`. For local testing, run `./scripts/test-update.sh`.

Manual `Dockerfile` changes also force a rebuild of the corresponding image, even when the parsed version tag already exists in GHCR.

## Workflow Relationship

```text
.github/config/images.yml
  -> .github/workflows/bump-base-images.yml
     -> updates image Dockerfile on main when a newer base image exists
     -> push to main
     -> explicit workflow_dispatch
        -> .github/workflows/build-caddy-cloudflare-geoip-ratelimit.yml
           -> uses .github/workflows/build-image.yml
              -> build/push image to GHCR

manual Dockerfile change on main
  -> .github/workflows/build-caddy-cloudflare-geoip-ratelimit.yml
     -> uses .github/workflows/build-image.yml
```
