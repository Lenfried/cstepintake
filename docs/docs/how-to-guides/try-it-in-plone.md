---
myst:
  html_meta:
    "description": "Submit and decide a test STEP application in a local Plone site"
    "property=og:description": "Submit and decide a test STEP application in a local Plone site"
    "property=og:title": "Try the application intake in Plone"
    "keywords": "Plone, c-step-intake, STEP, CSTEP, workflow"
---

# Try the application intake in Plone

1.  To start from a clean site, recreate it. This deletes all content.

    ```shell
    cd backend
    DELETE_EXISTING=1 make create-site
    ```

2.  Start the backend and frontend as described in the README, and log in as `admin` / `admin`.
3.  Add a **STEP Application** and click **Submit**. The automatic check runs, and the application moves to **Manual review**,
    where the eligibility panel shows the result. Every submission goes there, eligible or not.
4.  Accept, reject, or request more information as an admin, or as a user with the `ProgramDirector` role.
    Users with the `ProgramStaff` role can review applications and request more information, but cannot accept or reject them.
    If the [n8n webhook](n8n-webhook.md) is configured, each of these decisions is posted to n8n.

## Turn on CSTEP

CSTEP is off by default. To turn it on, open `portal_setup` at http://localhost:8080/Plone/portal_setup,
go to the **Import** tab, and import the profile "c_step_intake: CSTEP applications".
