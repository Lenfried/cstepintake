# c-step-intake

[![Built with Cookieplone](https://img.shields.io/badge/built%20with-Cookieplone-0083be.svg?logo=cookiecutter)](https://github.com/plone/cookieplone-templates/)
[![Black code style](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![CI](https://github.com/Lenfried/c-step-intake/actions/workflows/main.yml/badge.svg)](https://github.com/Lenfried/c-step-intake/actions/workflows/main.yml)

A new project using Plone 6.

## Quick Start

### Prerequisites

-   An [operating system](https://6.docs.plone.org/install/create-project-cookieplone.html#prerequisites-for-installation) that runs all the requirements mentioned.
-   [uv](https://6.docs.plone.org/install/create-project-cookieplone.html#uv)
-   [nvm](https://6.docs.plone.org/install/create-project-cookieplone.html#nvm)
-   [Node.js and pnpm](https://6.docs.plone.org/install/create-project.html#node-js) 24
-   [Make](https://6.docs.plone.org/install/create-project-cookieplone.html#make)
-   [Git](https://6.docs.plone.org/install/create-project-cookieplone.html#git)
-   [Docker](https://docs.docker.com/get-started/get-docker/) (optional)


### Installation

1.  Clone this repository, then change your working directory.

    ```shell
    git clone git@github.com:Lenfried/cstepintake.git c-step-intake
    cd c-step-intake
    ```

2.  Check out the intake add-on into `backend/sources` (see [Intake add-on](#intake-add-on)).
    This step is manual: nothing else fetches the add-on.

    ```shell
    git clone git@github.com:Lenfried/c-step-intake.git backend/sources/c-step-intake-core
    ```

3.  Install this code base.

    ```shell
    make install
    ```


### Fire Up the Servers

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

Voila! Your Plone site should be live and kicking!

## Intake add-on

The STEP/CSTEP application intake lives in a separate add-on,
[Lenfried/c-step-intake](https://github.com/Lenfried/c-step-intake) (Python package `c_step_intake`),
checked out at `backend/sources/c-step-intake-core`. That folder is gitignored, so the add-on's code is not in this repository.
[The intake add-on](./docs/docs/concepts/intake-add-on.md) explains how it is wired in and why.

> **Warning:** CI and the Docker image builds currently fail, because a fresh clone has no add-on checkout.
> `make build-images`, `make stack-create-site` and `make stack-start` do not work yet.
> See [Docker images and CI status](./docs/docs/reference/docker-and-ci-status.md) for the cause and the fix.

### What the add-on does

- **Content types:** STEP and CSTEP applications, with a review workflow. CSTEP is off by default.
- **Single-page forms:** every form section on one page instead of tabs ([details](./docs/docs/concepts/intake-add-on.md#single-page-application-forms)).
- **Eligibility panel:** staff see each submission checked against the STEP criteria ([details](./docs/docs/concepts/intake-add-on.md#eligibility-panel)).
- **School pick-list:** the STEP school is chosen from 1,998 NYSED secondary schools and stored in
  NYSED reporting format, `NAME  (COUNTY) BEDSCODE`. A "My school isn't listed" choice lets applicants type the name instead;
  staff should correct those before accepting, since they aren't in reporting format.
- **STEP acceptance criteria:** the form collects every condition, including a STEM statement and
  "ward of the State or a county". Each submission is checked against them for staff; nothing is accepted or rejected automatically.
- **Pre-filled fields:** name and email (and CSTEP's EMPLID, where the login provides it) start filled in from the applicant's account.
- **New-application email** to staff: [Set up the new-application email](./docs/docs/how-to-guides/new-application-email.md).
- **Staff-decision webhook:** accept, reject and request-more-information decisions are posted to n8n,
  which emails the student and staff: [Set up the staff-decision webhook to n8n](./docs/docs/how-to-guides/n8n-webhook.md).

To submit and decide a test application, see [Try the application intake in Plone](./docs/docs/how-to-guides/try-it-in-plone.md).
The add-on's own README and its `docs/` folder have the details.

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

### Local Stack Deployment

> **Note:** this does not work yet; see the warning under [Intake add-on](#intake-add-on).

Deploy a local Docker Compose environment that includes the following.

- Docker images for Backend and Frontend
- A stack with a Traefik router and a PostgreSQL database
- Accessible at [http://c-step-intake.localhost](http://c-step-intake.localhost)

Run the following commands in a shell session.

```shell
make stack-create-site
make stack-start
```

And... you're all set! Your Plone site is up and running locally!

## Documentation

More documentation is in [`docs/docs/`](./docs/docs/):

- [The intake add-on](./docs/docs/concepts/intake-add-on.md): how it is wired in, forms, eligibility panel
- [Try the application intake in Plone](./docs/docs/how-to-guides/try-it-in-plone.md)
- [Set up the new-application email](./docs/docs/how-to-guides/new-application-email.md)
- [Set up the staff-decision webhook to n8n](./docs/docs/how-to-guides/n8n-webhook.md)
- [Docker images and CI status](./docs/docs/reference/docker-and-ci-status.md)

## Project structure

This monorepo consists of the following distinct sections:

- **backend**: Houses the API and Plone installation, utilizing pip instead of buildout, and includes a policy package named c.step.intake.
- **frontend**: Contains the React (Volto) package.
- **devops**: Encompasses Docker stack, Ansible playbooks, and cache settings.
- **docs**: Scaffold for writing documentation for your project.

### Why this structure?

- All necessary codebases to run the site are contained within the repository (excluding existing add-ons for Plone and React).
- Specific GitHub Workflows are triggered based on changes in each codebase (refer to .github/workflows).
- Simplifies the creation of Docker images for each codebase.
- Demonstrates Plone installation/setup without buildout.

## Code quality assurance

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
| frontend | ESLint | Fixes most common frontend issues | [`frontend/.eslintrc.js`](./frontend/.eslintrc.js) |
| frontend | prettier | Format JS and Typescript code  | [`frontend/.prettierrc`](./frontend/.prettierrc) |
| frontend | Stylelint | Format Styles (css, less, sass)  | [`frontend/.stylelintrc`](./frontend/.stylelintrc) |

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
| frontend | ESLint | Checks JS / Typescript lint | [`frontend/.eslintrc.js`](./frontend/.eslintrc.js) |
| frontend | prettier | Check JS / Typescript formatting  | [`frontend/.prettierrc`](./frontend/.prettierrc) |
| frontend | Stylelint | Check Styles (css, less, sass) formatting  | [`frontend/.stylelintrc`](./frontend/.stylelintrc) |

Linters can be run individually within the `backend` or `frontend` folders.


## Internationalization

Generate translation files for Plone and Volto with ease:

```shell
make i18n
```

## Credits and acknowledgements

Generated using [Cookieplone (2.0.0)](https://github.com/plone/cookieplone) and [cookieplone-templates (db76a81)](https://github.com/plone/cookieplone-templates/commit/db76a81d89db7ed23d6873e5323bdbfc63cd6197) on 2026-09-26 18:54:55.210360. A special thanks to all contributors and supporters!
