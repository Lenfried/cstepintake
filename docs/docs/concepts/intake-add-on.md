---
myst:
  html_meta:
    "description": "How the STEP/CSTEP intake add-on is wired into c-step-intake"
    "property=og:description": "How the STEP/CSTEP intake add-on is wired into c-step-intake"
    "property=og:title": "The intake add-on"
    "keywords": "Plone, c-step-intake, add-on, mxdev, Volto"
---

# The intake add-on

The STEP/CSTEP application intake lives in a separate add-on,
[Lenfried/c-step-intake](https://github.com/Lenfried/c-step-intake) (Python package `c_step_intake`).
This project does not contain its code; it consumes it.

## How it is wired in

- **Checkout:** `backend/sources/c-step-intake-core`, on the add-on's `master` branch.
  `backend/sources/` is gitignored, so the add-on is not part of this repository
  (see "Why `backend/sources/` is gitignored" below).
- **Wiring:** [`backend/mx.ini`](https://github.com/Lenfried/cstepintake/blob/main/backend/mx.ini)
  registers it with mxdev (`vcs = fs`), and
  [`backend/pyproject.toml`](https://github.com/Lenfried/cstepintake/blob/main/backend/pyproject.toml)
  installs it in editable mode.
  The distribution is named `c-step-intake-core`, not `c-step-intake`, because Python treats
  `c-step-intake` and this project's `c.step.intake` as the same name.
- **Loading:** [`dependencies.zcml`](https://github.com/Lenfried/cstepintake/blob/main/backend/src/c/step/intake/dependencies.zcml)
  includes its ZCML, and
  [`metadata.xml`](https://github.com/Lenfried/cstepintake/blob/main/backend/src/c/step/intake/profiles/default/metadata.xml)
  installs `profile-c_step_intake:default` with this project's profile.
- **Volto:** the add-on's `frontend/` folder is the Volto add-on `volto-c-step-intake-core`.
  [`frontend/pnpm-workspace.yaml`](https://github.com/Lenfried/cstepintake/blob/main/frontend/pnpm-workspace.yaml)
  includes it from the same checkout.
  It is a dependency in [`frontend/package.json`](https://github.com/Lenfried/cstepintake/blob/main/frontend/package.json),
  where Volto's add-on registry resolves it.
  It is also listed under `addons` in
  [`volto-c-step-intake`](https://github.com/Lenfried/cstepintake/blob/main/frontend/packages/volto-c-step-intake/package.json).
  After cloning the add-on, run `make frontend-install` (or `pnpm install` in `frontend`) so the package is linked.

The add-on's own README and its `docs/` folder have the details of what it does.

## Single-page application forms

STEP and CSTEP add and edit forms show every section on one page, each under its own heading, instead of as tabs.
Two parts of the add-on do this:

- **Classic UI (:8080):** the add-on's `browser/forms.py` turns form tabbing off.
- **Volto (:3000):** Volto ignores that setting. It builds forms from the `@types` schema and draws tabs whenever
  there is more than one fieldset. `volto-c-step-intake-core` merges the fieldsets of these two types into one
  as the schema arrives, adding a heading field for each section. The backend schema is unchanged.

Other content types keep their tabs. The list of types is `config.settings.cStepIntakeSinglePageFormTypes`.

## Eligibility panel

Program staff (Manager, Site Administrator, ProgramDirector, ProgramStaff) see an **Eligibility check** panel
at the top of each submitted application in Volto. It summarises the latest automatic check:
each STEP acceptance condition as met / not met / undetermined, using the 2026-2027 NYSED income limits,
plus an overall conclusion and any missing answers. It never decides an application: every submission goes to staff review.
Applicants, and drafts, show no panel. The data comes from the add-on's `GET <application>/@eligibility` endpoint.
See the add-on's `docs/eligibility_rules.md` for the criteria.

## Why `backend/sources/` is gitignored

`/sources/` is ignored by the Cookieplone project template, not by a choice made here.
It is where mxdev puts packages listed in `mx.ini` that come from somewhere other than PyPI.
Each of those is a separate git repository with its own history and its own owners,
so this repository records **which** package to use (in `mx.ini`), not a copy of its code.
Committing the folder would duplicate the add-on's code here, and git cannot track a folder
that contains its own `.git` as ordinary files.

That works when `mx.ini` tells mxdev where to get the package. Here it doesn't:
the entry uses `vcs = fs`, which means "the package is already on disk, don't fetch it".
So the only thing that puts the add-on in `backend/sources/` is the manual clone in the
README's Installation steps. It is also why CI and the Docker images fail; see
[Docker images and CI status](../reference/docker-and-ci-status.md).
