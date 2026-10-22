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

    ```shell
    git clone -b rename-to-c-step-intake git@github.com:Lenfried/c-step-intake.git backend/sources/c-step-intake-core
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

- **Checkout:** `backend/sources/c-step-intake-core`, on the `rename-to-c-step-intake` branch.
  `backend/sources/` is gitignored, so the add-on is not part of this repository.
- **Wiring:** [`backend/mx.ini`](./backend/mx.ini) registers it with mxdev (`vcs = fs`), and
  [`backend/pyproject.toml`](./backend/pyproject.toml) installs it in editable mode.
- **Loading:** [`dependencies.zcml`](./backend/src/c/step/intake/dependencies.zcml) includes its ZCML, and
  [`metadata.xml`](./backend/src/c/step/intake/profiles/default/metadata.xml) installs `profile-c_step_intake:default` with this project's profile.
- **Volto:** the add-on's `frontend/` folder is the Volto add-on `volto-c-step-intake-core`.
  [`frontend/pnpm-workspace.yaml`](./frontend/pnpm-workspace.yaml) includes it from the same checkout.
  It is a dependency in [`frontend/package.json`](./frontend/package.json), where Volto's add-on registry resolves it.
  It is also listed under `addons` in [`volto-c-step-intake`](./frontend/packages/volto-c-step-intake/package.json).
  After cloning the add-on, run `make frontend-install` (or `pnpm install` in `frontend`) so the package is linked.

### Single-page application forms

STEP and CSTEP add and edit forms show every section on one page, each under its own heading, instead of as tabs.
Two parts of the add-on do this:

- **Classic UI (:8080):** the add-on's `browser/forms.py` turns form tabbing off.
- **Volto (:3000):** Volto ignores that setting. It builds forms from the `@types` schema and draws tabs whenever
  there is more than one fieldset. `volto-c-step-intake-core` merges the fieldsets of these two types into one
  as the schema arrives, adding a heading field for each section. The backend schema is unchanged.

Other content types keep their tabs. The list of types is `config.settings.cStepIntakeSinglePageFormTypes`.

The `rename-to-c-step-intake` branch is upstream `master` plus these project-specific changes:

- The distribution is named `c-step-intake-core`, because `c-step-intake` would clash with this project's `c.step.intake` package.
- It registers through `plone.autoinclude` with `module = c_step_intake`.
- `global_allow` is `True` for STEP and CSTEP applications, so they can be added anywhere.
- STEP `cumulative_gpa` has no upper bound.

### Update the add-on

Merge upstream `master` into the branch, keeping the changes above:

```shell
cd backend/sources/c-step-intake-core
git fetch origin
git merge origin/master
git push origin rename-to-c-step-intake
```

Code changes take effect without reinstalling. If upstream adds GenericSetup files, such as a new
`registry.xml`, existing sites need that import step re-run in `portal_setup`, or the site recreated.

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
3.  Add a **STEP Application** and click **Submit**. The automatic check runs, and the application moves to **Manual review**.
    No accept or reject rules are defined yet, so every complete application goes there.
4.  Accept, reject, or request more information as an admin, or as a user with the `ProgramDirector` role.
    Users with the `ProgramStaff` role can review applications but cannot decide them.

CSTEP is off by default. To turn it on, open `portal_setup` at http://localhost:8080/Plone/portal_setup,
go to the **Import** tab, and import the profile "c_step_intake: CSTEP applications".

### New-application email

The add-on sends one email when an application is **first submitted**.
Saving drafts, resubmitting, and staff transitions do not send one.
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
