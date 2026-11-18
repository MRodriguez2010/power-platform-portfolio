# Platform design

## Working within the tenant we had

Lockton's Microsoft 365 tenant did not use Azure data lakes, Synapse pools, Dataverse or any other managed database for this work. That ruled out the usual "app + database + pipeline" pattern. Every solution in this portfolio uses only standard Microsoft 365 components:

| Need | Usual enterprise choice | What was used | Trade-off accepted |
|---|---|---|---|
| System of record | Dataverse / SQL | Excel tables in SharePoint | Row limits and concurrency need care; flows wait briefly after writes before reading |
| Business rules | App logic / stored procedures | Power Automate expressions and conditions | Rules live in the flow, so changes go through the flow owner |
| Joining datasets | SQL view / data pipeline | Power Query | Refresh-time merge rather than live |
| Reporting | Power BI on a warehouse | Power BI on SharePoint Excel | Cloud source, so no gateway needed |
| Access control | Database roles | SharePoint site and folder permissions | Permissions managed per folder |
| Notifications | Third-party messaging tools | Outlook connector | Same mailbox controls as normal email |

## Security model

- Data stays in the tenant. Nothing is copied to a third-party service, which was the deciding factor for HRO data containing protected health information.
- Separate folders per audience (developers, certifiers, program leadership, HR), each with its own permissions.
- Flows run under named connections owned by the team responsible for the process.
- Reporting tables (the approved master) are derived, so they can always be rebuilt from the source tables.

## Enabling leadership

Before the certification processes could exist, leadership had to move off local Excel files. The director wanted the Alteryx Champs to run the program, so Miguel, as the team's Power Platform subject-matter expert, walked leadership through SharePoint and Power Platform practices: shared libraries instead of emailed copies, forms writing directly to tables, and folder-level permissions. Having one secure place with access set per folder is what got buy-in.

## Reliability practices used in the demos

- **Idempotent runs.** A record is only created if its ID doesn't already exist, and emails go out only for records created in the current run.
- **No blind retries on sends.** Email actions have retries turned off; an uncertain outcome is logged and held rather than risking a duplicate message.
- **Receipts.** Each run writes a JSON receipt per record to SharePoint, and the demo repositories check those receipts against an expected baseline (`validation/`).
- **Synthetic data only.** Fixtures use invented people and placeholder departments.
