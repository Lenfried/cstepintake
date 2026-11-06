---
myst:
  html_meta:
    "description": "Configure and test the new-application email to staff"
    "property=og:description": "Configure and test the new-application email to staff"
    "property=og:title": "Set up the new-application email"
    "keywords": "Plone, c-step-intake, email, SMTP, Mailpit"
---

# Set up the new-application email

The add-on sends one email when an application is **first submitted**.
Saving drafts, resubmitting, and staff transitions do not send one
(staff decisions go to n8n instead; see [Set up the staff-decision webhook to n8n](n8n-webhook.md)).

## Change the recipient

The recipient is the registry record `c_step_intake.notification_email`.
Its default is `cstep-n8n@mailings.york.cuny.edu`; leave it empty to turn notifications off.
Change it at http://localhost:8080/Plone/portal_registry (filter on `c_step_intake`).
Volto's control panel does not include the registry editor.

## Configure mail

Plone needs both an SMTP server and a site "From" address.
If either is missing, the send fails. The submission still goes through, and the error appears only in the backend log.

## Catch mail locally

For local development, catch mail with [Mailpit](https://mailpit.axllent.org/) (a standalone binary, not a Python package):

```shell
mailpit --smtp 127.0.0.1:1025 --listen 127.0.0.1:8025
```

In **Site Setup → Mail** (http://localhost:8080/Plone/@@mail-controlpanel), set:

- **SMTP server:** `localhost`
- **Port:** `1025`
- **Site 'From' address:** any address, for example `noreply@example.com`

Messages then appear at http://localhost:8025 instead of being delivered.
