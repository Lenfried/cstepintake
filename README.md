# c-step-intake 🚀

[![Built with Cookieplone](https://img.shields.io/badge/built%20with-Cookieplone-0083be.svg?logo=cookiecutter)](https://github.com/plone/cookieplone-templates/)
[![Black code style](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![CI](https://github.com/Lenfried/c-step-intake/actions/workflows/main.yml/badge.svg)](https://github.com/Lenfried/c-step-intake/actions/workflows/main.yml)

A new project using Plone 6.

## Quick Start 🏁

### Prerequisites ✅

-   An [operating system](https://6.docs.plone.org/install/create-project-cookieplone.html#prerequisites-for-installation) that runs all the requirements mentioned.
-   [uv](https://6.docs.plone.org/install/create-project-cookieplone.html#uv)
-   [nvm](https://6.docs.plone.org/install/create-project-cookieplone.html#nvm)
-   [Node.js and pnpm](https://6.docs.plone.org/install/create-project.html#node-js) 24
-   [Make](https://6.docs.plone.org/install/create-project-cookieplone.html#make)
-   [Git](https://6.docs.plone.org/install/create-project-cookieplone.html#git)
-   [Docker](https://docs.docker.com/get-started/get-docker/) (optional)


### Installation 🔧

1.  Clone this repository, then change your working directory.

    ```shell
    git clone git@github.com:Lenfried/cstepintake.git c-step-intake
    cd c-step-intake
    ```

2.  Check out the intake add-on into `backend/sources` (see [Intake add-on](#intake-add-on-)).
    This step is manual: nothing else fetches the add-on.

    ```shell
    git clone git@github.com:Lenfried/c-step-intake.git backend/sources/c-step-intake-core
    ```

3.  Install this code base.

    ```shell
    make install
    ```


### Fire Up the Servers 🔥

1.  Create a new Plone site on your first run.

    ```shell
    make backend-create-site
    ```

2.  Start the backend at http://localhost:8080/.

    ```shell
    make backend-start
    ```

3.  In a new shell session, start the frontend at http://localhost:3000/.

    ```shell
    make frontend-start
    ```

Voila! Your Plone site should be live and kicking! 🎉

## Intake add-on 📝

The STEP/CSTEP application intake lives in a separate add-on,
[Lenfried/c-step-intake](https://github.com/Lenfried/c-step-intake) (Python package `c_step_intake`).
This project consumes it as follows:

- **Checkout:** `backend/sources/c-step-intake-core`, on the add-on's `master` branch.
  `backend/sources/` is gitignored, so the add-on is not part of this repository
  (see [Why `backend/sources/` is gitignored](#why-backendsources-is-gitignored)).
- **Wiring:** [`backend/mx.ini`](./backend/mx.ini) registers it with mxdev (`vcs = fs`), and
  [`backend/pyproject.toml`](./backend/pyproject.toml) installs it in editable mode.
  The distribution is named `c-step-intake-core`, not `c-step-intake`, because Python treats
  `c-step-intake` and this project's `c.step.intake` as the same name.
- **Loading:** [`dependencies.zcml`](./backend/src/c/step/intake/dependencies.zcml) includes its ZCML, and
  [`metadata.xml`](./backend/src/c/step/intake/profiles/default/metadata.xml) installs `profile-c_step_intake:default` with this project's profile.
- **Volto:** the add-on's `frontend/` folder is the Volto add-on `volto-c-step-intake-core`.
  [`frontend/pnpm-workspace.yaml`](./frontend/pnpm-workspace.yaml) includes it from the same checkout.
  It is a dependency in [`frontend/package.json`](./frontend/package.json), where Volto's add-on registry resolves it.
  It is also listed under `addons` in [`volto-c-step-intake`](./frontend/packages/volto-c-step-intake/package.json).
  After cloning the add-on, run `make frontend-install` (or `pnpm install` in `frontend`) so the package is linked.

### What the add-on does

- **Content types:** STEP and CSTEP applications, with a review workflow. CSTEP is off by default.
- **Single-page forms** and the staff **eligibility panel** (both below).
- **School pick-list:** the STEP school is chosen from 1,998 NYSED secondary schools and stored in
  NYSED reporting format, `NAME  (COUNTY) BEDSCODE`. A "My school isn't listed" choice lets applicants type the name instead;
  staff should correct those before accepting, since they aren't in reporting format.
- **STEP acceptance criteria:** the form collects every condition, including a STEM statement and
  "ward of the State or a county". Each submission is checked against them for staff; nothing is accepted or rejected automatically.
- **Pre-filled fields:** name and email (and CSTEP's EMPLID, where the login provides it) start filled in from the applicant's account.
- **New-application email** to staff (below).
- **Staff-decision webhook:** accept, reject and request-more-information decisions are posted to n8n,
  which emails the student (below).

The add-on's own README and its `docs/` folder have the details.

### Single-page application forms

STEP and CSTEP add and edit forms show every section on one page, each under its own heading, instead of as tabs.
Two parts of the add-on do this:

- **Classic UI (:8080):** the add-on's `browser/forms.py` turns form tabbing off.
- **Volto (:3000):** Volto ignores that setting. It builds forms from the `@types` schema and draws tabs whenever
  there is more than one fieldset. `volto-c-step-intake-core` merges the fieldsets of these two types into one
  as the schema arrives, adding a heading field for each section. The backend schema is unchanged.

Other content types keep their tabs. The list of types is `config.settings.cStepIntakeSinglePageFormTypes`.

### Eligibility panel

Program staff (Manager, Site Administrator, ProgramDirector, ProgramStaff) see an **Eligibility check** panel
at the top of each submitted application in Volto. It summarises the latest automatic check:
each STEP acceptance condition as met / not met / undetermined, using the 2026-2027 NYSED income limits,
plus an overall conclusion and any missing answers. It never decides an application: every submission goes to staff review.
Applicants, and drafts, show no panel. The data comes from the add-on's `GET <application>/@eligibility` endpoint.
See the add-on's `docs/eligibility_rules.md` for the criteria.

### Update the add-on

Pull the add-on's `master`:

```shell
cd backend/sources/c-step-intake-core
git switch master
git pull
```

Work on the add-on happens on a branch in its own repository, merged into `master` by pull request.
Everything this project needs is in `master`, so no project-specific branch is kept.

Code changes take effect without reinstalling. If upstream adds GenericSetup files, such as a new
`registry.xml`, existing sites need that import step re-run in `portal_setup`, or the site recreated.
When the add-on ships an upgrade step (the profile version in its `metadata.xml` goes up, as with 1001 for the n8n webhook),
run it from Site Setup → Add-ons instead.

### Test the add-on

Run the add-on's tests from `backend`:

```shell
.venv/bin/python -m pytest sources/c-step-intake-core/src/c_step_intake/tests -q
```

`make backend-test` runs only this project's tests in `backend/tests`, not the add-on's.

Run the Volto add-on's tests from `frontend`:

```shell
pnpm --filter volto-c-step-intake-core exec vitest run
```

### Try it in Plone

1.  To start from a clean site, recreate it. This deletes all content.

    ```shell
    cd backend
    DELETE_EXISTING=1 make create-site
    ```

2.  Start the backend and frontend as described above, and log in as `admin` / `admin`.
3.  Add a **STEP Application** and click **Submit**. The automatic check runs, and the application moves to **Manual review**,
    where the eligibility panel shows the result. Every submission goes there, eligible or not.
4.  Accept, reject, or request more information as an admin, or as a user with the `ProgramDirector` role.
    Users with the `ProgramStaff` role can review applications but cannot decide them.

CSTEP is off by default. To turn it on, open `portal_setup` at http://localhost:8080/Plone/portal_setup,
go to the **Import** tab, and import the profile "c_step_intake: CSTEP applications".

### New-application email

The add-on sends one email when an application is **first submitted**.
Saving drafts, resubmitting, and staff transitions do not send one
(staff decisions go to n8n instead; see [Staff-decision webhook to n8n](#staff-decision-webhook-to-n8n-)).
The recipient is the registry record `c_step_intake.notification_email`.
Its default is `cstep-n8n@mailings.york.cuny.edu`; leave it empty to turn notifications off.
Change it at http://localhost:8080/Plone/portal_registry (filter on `c_step_intake`).
Volto's control panel does not include the registry editor.

Plone needs both an SMTP server and a site "From" address.
If either is missing, the send fails. The submission still goes through, and the error appears only in the backend log.

For local development, catch mail with [Mailpit](https://mailpit.axllent.org/) (a standalone binary, not a Python package):

```shell
mailpit --smtp 127.0.0.1:1025 --listen 127.0.0.1:8025
```

In **Site Setup → Mail** (http://localhost:8080/Plone/@@mail-controlpanel), set:

- **SMTP server:** `localhost`
- **Port:** `1025`
- **Site 'From' address:** any address, for example `noreply@example.com`

Messages then appear at http://localhost:8025 instead of being delivered.

### Staff-decision webhook to n8n 🔔

When staff **accept**, **reject**, or **request more information** on an application in **Manual review**,
the add-on posts an event to an n8n webhook, and the n8n workflow emails the student.
The automatic check's own transitions are not posted, only human decisions.
The event carries no applicant details; n8n looks the student up through the REST API.
It is sent after the change is saved, and if n8n is down the decision still goes through and the error is logged.
The add-on's README has the payload format.

Three pieces have to agree:

| Piece | Where | Value |
|---|---|---|
| Webhook URL | Plone registry record `c_step_intake.n8n_webhook_url` | The n8n workflow's **production** webhook URL. Empty turns the webhook off. |
| Shared secret (Plone) | Environment variable `C_STEP_INTAKE_N8N_WEBHOOK_TOKEN` on the backend | A random token, for example from `openssl rand -hex 32` |
| Shared secret (n8n) | n8n Header Auth credential | Header name `X-STEP-Token`, same token as above |

The token is never committed. Where it is set:

- **Docker Compose:** [`docker-compose.yml`](./docker-compose.yml) and
  [`devops/stacks/c-step-intake.example.com.yml`](./devops/stacks/c-step-intake.example.com.yml) pass the variable
  to the backend container. Locally, its value comes from a `.env` file next to `docker-compose.yml`, which is git-ignored.
- **Backend without Docker:** `make backend-start` does not read `.env`. Load it first:

  ```shell
  set -a; source .env; set +a
  make backend-start
  ```

- **GitHub Actions:** the repository has an Actions secret `C_STEP_INTAKE_N8N_WEBHOOK_TOKEN`.
  No workflow deploys yet, so nothing uses it; a future deploy job would pass it to `docker stack deploy`.

**The n8n workflow** is in [`n8n/application-decision-workflow.json`](./n8n/application-decision-workflow.json).
Import it with **Workflows → Import from File**, then:

1. Create three credentials and select them on the nodes: **Plone webhook token** (Header Auth, as above),
   **Plone service account** (Basic Auth, a Plone user that can only view applications), and **SMTP account**.
2. Set the Plone address in **Fetch application from Plone** (it ships as `http://localhost:8080/Plone`)
   and the sender address in the three email nodes.
3. Test it, activate it, and copy its production webhook URL into the registry record above.

**Turning it on for an existing site:** run the add-on's **1001 upgrade step** in Site Setup → Add-ons,
which adds the registry record, then set the three pieces above.

**Network:** the Plone backend must be able to reach n8n, and n8n must be able to reach Plone's REST API.
The current n8n instance is only reachable over VPN.

**Handing off:** give the token through a password manager, not email or chat, or have the new owner
generate a fresh one and set it in both places.

### Why `backend/sources/` is gitignored

`/sources/` is ignored by the Cookieplone project template, not by a choice made here.
It is where mxdev puts packages listed in `mx.ini` that come from somewhere other than PyPI.
Each of those is a separate git repository with its own history and its own owners,
so this repository records **which** package to use (in `mx.ini`), not a copy of its code.
Committing the folder would duplicate the add-on's code here, and git cannot track a folder
that contains its own `.git` as ordinary files.

That works when `mx.ini` tells mxdev where to get the package. Here it doesn't:
the entry uses `vcs = fs`, which means "the package is already on disk, don't fetch it".
So the only thing that puts the add-on in `backend/sources/` is the manual clone in
[Installation](#installation-), step 2.

### Why the Docker images and CI don't build ⚠️

Anything that starts from a fresh clone of this repository has no `backend/sources/`, so it has no add-on.

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

**How to fix it** (not done yet):

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

Until then, `make build-images`, `make stack-create-site` and `make stack-start` below do not work:
the frontend image cannot build on any machine, and the backend image cannot build from a fresh clone.

### Local Stack Deployment 📦

Deploy a local Docker Compose environment that includes the following.

- Docker images for Backend and Frontend 🖼️
- A stack with a Traefik router and a PostgreSQL database 🗃️
- Accessible at [http://c-step-intake.localhost](http://c-step-intake.localhost) 🌐

Run the following commands in a shell session.

```shell
make stack-create-site
make stack-start
```

And... you're all set! Your Plone site is up and running locally! 🚀

## Project structure 🏗️

This monorepo consists of the following distinct sections:

- **backend**: Houses the API and Plone installation, utilizing pip instead of buildout, and includes a policy package named c.step.intake.
- **frontend**: Contains the React (Volto) package.
- **devops**: Encompasses Docker stack, Ansible playbooks, and cache settings.
- **docs**: Scaffold for writing documentation for your project.

### Why this structure? 🤔

- All necessary codebases to run the site are contained within the repository (excluding existing add-ons for Plone and React).
- Specific GitHub Workflows are triggered based on changes in each codebase (refer to .github/workflows).
- Simplifies the creation of Docker images for each codebase.
- Demonstrates Plone installation/setup without buildout.

## Code quality assurance 🧐

To check your code against quality standards, run the following shell command.

```shell
make check
```

### Format the codebase

To format and rewrite the code base, ensuring it adheres to quality standards, run the following shell command.

```shell
make format
```

| Section | Tool | Description | Configuration |
| --- | --- | --- | --- |
| backend | Ruff | Python code formatting, imports sorting  | [`backend/pyproject.toml`](./backend/pyproject.toml) |
| backend | `zpretty` | XML and ZCML formatting  | -- |
| frontend | ESLint | Fixes most common frontend issues | [`frontend/.eslintrc.js`](.frontend/.eslintrc.js) |
| frontend | prettier | Format JS and Typescript code  | [`frontend/.prettierrc`](.frontend/.prettierrc) |
| frontend | Stylelint | Format Styles (css, less, sass)  | [`frontend/.stylelintrc`](.frontend/.stylelintrc) |

Formatters can also be run within the `backend` or `frontend` folders.

### Linting the codebase
or `lint`:

 ```shell
make lint
```

| Section | Tool | Description | Configuration |
| --- | --- | --- | --- |
| backend | Ruff | Checks code formatting, imports sorting  | [`backend/pyproject.toml`](./backend/pyproject.toml) |
| backend | Pyroma | Checks Python package metadata  | -- |
| backend | check-python-versions | Checks Python version information  | -- |
| backend | `zpretty` | Checks XML and ZCML formatting  | -- |
| frontend | ESLint | Checks JS / Typescript lint | [`frontend/.eslintrc.js`](.frontend/.eslintrc.js) |
| frontend | prettier | Check JS / Typescript formatting  | [`frontend/.prettierrc`](.frontend/.prettierrc) |
| frontend | Stylelint | Check Styles (css, less, sass) formatting  | [`frontend/.stylelintrc`](.stylelintrc) |

Linters can be run individually within the `backend` or `frontend` folders.


## Internationalization 🌐

Generate translation files for Plone and Volto with ease:

```shell
make i18n
```

## Credits and acknowledgements 🙏

Generated using [Cookieplone (2.0.0)](https://github.com/plone/cookieplone) and [cookieplone-templates (db76a81)](https://github.com/plone/cookieplone-templates/commit/db76a81d89db7ed23d6873e5323bdbfc63cd6197) on 2026-09-26 18:54:55.210360. A special thanks to all contributors and supporters!
