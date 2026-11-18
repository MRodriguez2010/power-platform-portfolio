# Alteryx governance: operating detail

The [README](../README.md#2-alteryx-governance-operational-flow) shows the end-to-end flow. This page covers the statuses, data, rules and notifications behind it, and maps each to the demo that reproduces it.

## Why governance was needed

Alteryx was being rolled out across divisions. Two shared resources needed protection:

1. **Alteryx Server.** Scheduled workflows run unattended on shared infrastructure. A workflow that depends on a tool the server can't run (for example, one that needs a local file or desktop-only connector) fails there and wastes certifier time.
2. **The shared macro store.** A published macro is reused by everyone, so a defect spreads.

Certification gave both a single, repeatable path, and its records became the program's measure of adoption.

## Request lifecycle and statuses

| Status | Set when | Who acts next | Applies to |
|---|---|---|---|
| `NeedsInformation` | A required field is blank, or the department has no recertification rule | Developer fixes and resubmits | Workflow, macro |
| `LocalOnly` | The developer checked a tool that cannot run on Server | None; the workflow keeps running locally | Workflow only |
| `PendingReview` | Validation passed (and, for workflows, the server gate passed) | Alteryx Champs + Aimpoint; Aimpoint schedules the meeting | Workflow, macro |
| `Approved` | Certifier records Approve | Workflow scheduled on Server / macro published | Workflow, macro |
| `Denied` | Certifier records Deny | Developer may revise and resubmit | Workflow, macro |
| `Postponed` | Certifier records Postpone | Aimpoint reschedules | Workflow, macro |

Status is computed from the two tables each time the flow runs, never typed in by hand. That way the request table, the decision table and the approved master can't disagree.

## Rules

**Server gate (workflows only).** The checklist on the form lists *only* tools that are unsupported on Server. Selecting any of them sets `LocalOnly`. Using a checklist of excluded tools, instead of asking "is this server-ready?", turns a judgment call into a fact the developer can answer.

**Recertification date.** Calculated at intake from the submission date and the department's policy:

| Policy | Recertify by | Demo department |
|---|---|---|
| Calendar year | December 31 of the submission year | Operations |
| Rolling | 12 months after submission | Analytics |

Real department names and assignments were the employer's. The demo uses two placeholder departments to show both rules.

## Data model

All tables are Excel tables in a SharePoint document library, which was the approved storage. No Dataverse or Azure database was used.

**Requests** (written by the developer's form)

| Column | Purpose |
|---|---|
| RequestId, Revision | Identity; revision allows resubmission |
| Kind | Workflow or Macro |
| Submitter, Division, Department | Ownership and routing; department drives the recertification rule |
| Description | What the asset does |
| SelectedTools | Checked items from the excluded-tools list |
| SubmittedDate | Start of the recertification clock |

**Decisions** (written by the Aimpoint certifier's form)

| Column | Purpose |
|---|---|
| DecisionId | Identity of the decision |
| RequestId, Revision | Link to the request |
| Decision | Approve / Deny / Postpone |
| Certifier, Reason | Accountability and feedback |

**Approved master** (Power Query output)

Requests joined to Decisions on RequestId + Revision, filtered to `Approved`, with the computed recertify-by date. This is the table the director's ROI tracker referenced. See [`approved-master-references.pq`](https://github.com/MRodriguez2010/power-platform-workflow-certification-demo/blob/main/power-query/approved-master-references.pq).

## Notifications

| Event | Recipient in production | Content |
|---|---|---|
| `PendingReview` | Alteryx Champs + Aimpoint | Certification request: submitter, division/department, description, declared tools, recertify-by date if approved |
| `LocalOnly` | Developer | Not eligible for Server; may keep running locally |
| `NeedsInformation` | Developer | What to fix before resubmitting |
| Approved | Developer | Outcome and recertify-by date |
| Denied / Postponed | Developer | Outcome and next step |

In the current demo, all of these go to one test inbox with a `[DEMO]` subject and a line naming the real recipient. A run only emails about records created in that run, so re-running a flow doesn't resend old notices.

## Original vs. demo

| Original process | Demo reproduction |
|---|---|
| SharePoint form for developers | Flow manual-trigger inputs (form fields as trigger inputs) |
| Aimpoint certifier survey | Same flow's decision inputs, written to a separate table |
| Aimpoint sends meeting request | Not automated, matching the original; the email notice is the hand-off |
| Approved macros to SharePoint folder, later into the Alteryx platform | Approved macro publishes a labeled sample text file to a SharePoint folder |
| Approved master feeding the director's ROI tracker | Approved master in Power Query; the ROI tracker itself is not reproduced |

Demo repositories: [workflow certification](https://github.com/MRodriguez2010/power-platform-workflow-certification-demo) · [macro certification](https://github.com/MRodriguez2010/power-platform-macro-certification-demo).
