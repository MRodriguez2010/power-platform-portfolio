# Current flow definitions (v2, November 18, 2026)

These are the Power Automate action and trigger definitions running in the live demos. They add email notifications and the Power App trigger to the earlier preview-only versions in the individual demo repositories.

| File | Flow |
|---|---|
| `workflow-*.json` | Workflow server certification |
| `macro-*.json` | Macro store certification |
| `survey-*.json` | Event survey (Power Apps trigger, response back to the app) |
| `mail-*.json` | Conditional mail merge (optional `RESET` input re-arms sample recipients) |
| `survey-app-controls.yaml` | Canvas app controls: Email, Name, 1–5 rating, feedback, Submit |

Tenant-specific values are replaced with placeholders: `contoso.sharepoint.com/sites/PowerPlatformDemo`, `<DOCUMENT_LIBRARY_DRIVE_ID>`, `<SITE_GROUP_ID>`, and `demo-inbox@example.com` for the single test inbox. To import into another tenant, use the import package and `scripts/configure_import.py` in the matching demo repository, then paste these definitions in or rebuild the actions in the designer.

Every email action has retries turned off, and the subject is tagged `[DEMO]`.
