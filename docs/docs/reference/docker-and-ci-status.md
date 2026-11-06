---
myst:
  html_meta:
    "description": "Why the Docker images and CI do not build, and how to fix it"
    "property=og:description": "Why the Docker images and CI do not build, and how to fix it"
    "property=og:title": "Docker images and CI status"
    "keywords": "Plone, c-step-intake, Docker, CI, mxdev"
---

# Docker images and CI status

Anything that starts from a fresh clone of this repository has no `backend/sources/`, so it has no add-on
(see [The intake add-on](../concepts/intake-add-on.md) for why).

## What fails

- **CI:** every backend job has failed since the add-on was wired in (2026-10-19).
  They stop at the same step:
  `Directory 'sources/c-step-intake-core' for package 'c-step-intake-core' doesn't exist.`
  The image-build job never runs, because the backend jobs fail first.
- **Backend image** (`backend/Dockerfile`): the build runs `mxdev -c mx.ini`, then `uv sync`,
  both of which need `sources/c-step-intake-core`. On a fresh clone it fails the same way as CI.
  On a machine that has the checkout it can succeed, because the build context is `backend/`
  and `backend/.dockerignore` does not exclude `sources/`; the image then contains whatever
  branch that checkout happens to be on.
- **Frontend image** (`frontend/Dockerfile`): fails everywhere, including on a machine with the checkout.
  The Volto add-on is linked from `../backend/sources/c-step-intake-core/frontend`, outside the
  `frontend/` build context, and the Dockerfile copies only `packages/volto-c-step-intake` and the
  workspace files. `pnpm install` then cannot find the `volto-c-step-intake-core` dependency.

Until this is fixed, `make build-images`, `make stack-create-site` and `make stack-start` do not work:
the frontend image cannot build on any machine, and the backend image cannot build from a fresh clone.

## How to fix it

Not done yet.

1. **Backend:** change the `mx.ini` entry from `vcs = fs` to a git source, as in the commented examples in that file:

   ```ini
   [c-step-intake-core]
   url = https://github.com/Lenfried/c-step-intake.git
   branch = master
   ```

   mxdev then clones the add-on into `sources/` during `make install`, CI and the image build,
   and the manual clone step goes away.
2. **Frontend:** the image build must get the Volto add-on into the `frontend/` build context,
   either by fetching it during the build (for example cloning the add-on repository and copying its `frontend/` folder
   into `packages/`) or by publishing `volto-c-step-intake-core` to npm and depending on a version.
   Local development can keep the link to the backend checkout.
