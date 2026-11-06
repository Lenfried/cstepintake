---
myst:
  html_meta:
    "description": "Connect Plone staff decisions to the n8n workflow that emails students"
    "property=og:description": "Connect Plone staff decisions to the n8n workflow that emails students"
    "property=og:title": "Set up the staff-decision webhook to n8n"
    "keywords": "Plone, c-step-intake, n8n, webhook"
---

# Set up the staff-decision webhook to n8n

When staff **accept**, **reject**, or **request more information** on an application in **Manual review**,
the add-on posts an event to an n8n webhook, and the n8n workflow emails the student.
The automatic check's own transitions are not posted, only human decisions.
The event carries no applicant details; n8n looks the student up through the REST API.
It is sent after the change is saved, and if n8n is down the decision still goes through and the error is logged.
The add-on's README has the payload format.

## The three settings

Three pieces have to agree:

| Piece | Where | Value |
|---|---|---|
| Webhook URL | Plone registry record `c_step_intake.n8n_webhook_url` | The n8n workflow's **production** webhook URL. Empty turns the webhook off. |
| Shared secret (Plone) | Environment variable `C_STEP_INTAKE_N8N_WEBHOOK_TOKEN` on the backend | A random token, for example from `openssl rand -hex 32` |
| Shared secret (n8n) | n8n Header Auth credential | Header name `X-STEP-Token`, same token as above |

## Where the token is set

The token is never committed.

- **Docker Compose:** [`docker-compose.yml`](https://github.com/Lenfried/cstepintake/blob/main/docker-compose.yml) and
  [`devops/stacks/c-step-intake.example.com.yml`](https://github.com/Lenfried/cstepintake/blob/main/devops/stacks/c-step-intake.example.com.yml)
  pass the variable to the backend container.
  Locally, its value comes from a `.env` file next to `docker-compose.yml`, which is git-ignored.
- **Backend without Docker:** `make backend-start` does not read `.env`. Load it first:

  ```shell
  set -a; source .env; set +a
  make backend-start
  ```

- **GitHub Actions:** the repository has an Actions secret `C_STEP_INTAKE_N8N_WEBHOOK_TOKEN`.
  No workflow deploys yet, so nothing uses it; a future deploy job would pass it to `docker stack deploy`.

## Import the n8n workflow

The workflow is in [`n8n/application-decision-workflow.json`](https://github.com/Lenfried/cstepintake/blob/main/n8n/application-decision-workflow.json).
Import it with **Workflows → Import from File**, then:

1. Create three credentials and select them on the nodes: **Plone webhook token** (Header Auth, as above),
   **Plone service account** (Basic Auth, a Plone user that can only view applications), and **SMTP account**.
2. Set the Plone address in **Fetch application from Plone** (it ships as `http://localhost:8080/Plone`)
   and the sender address in the three email nodes.
3. Test it, activate it, and copy its production webhook URL into the registry record above.

## Turn it on for an existing site

Run the add-on's **1001 upgrade step** in Site Setup → Add-ons, which adds the registry record,
then set the three pieces above.

## Network

The Plone backend must be able to reach n8n, and n8n must be able to reach Plone's REST API.
The current n8n instance is only reachable over VPN.

## Handing off

Give the token through a password manager, not email or chat, or have the new owner
generate a fresh one and set it in both places.
