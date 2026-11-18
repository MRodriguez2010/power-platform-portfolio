# HR automations: operating notes

## Event surveys

**Stakeholders.** HR event organizers (owners), attendees (respondents), an HR reviewer for flagged comments, and anyone reading the Power BI dashboard.

**Flow.** Survey response → validate rating → check the feedback against a keyword dictionary → write to the responses table in SharePoint → email the reviewer if flagged. Power BI reads the table directly.

**Routes**

| Route | Meaning |
|---|---|
| `InvalidRating` | Rating missing or outside 1–5; stored but excluded from scoring |
| `CriticalFeedbackReview` | Feedback contains a dictionary keyword (demo list: *complaint, unsafe, harassment*); reviewer emailed |
| `Accepted` | Normal response |

**Why no gateway.** The dataset is an Excel file in SharePoint Online, a cloud source, so the Power BI service refreshes it without an on-premises data gateway. A gateway would only be needed for on-premises or local files.

**Known limitation.** Keyword matching flags "no complaint" as well as "complaint". It is a triage aid that sends comments to a human, not sentiment analysis.

**Current demo.** A canvas Power App collects Email, Name, a 1–5 star rating and open feedback. It calls the flow, then shows a reference number and whether the comment was flagged. The respondent gets a thank-you email; the reviewer gets an alert for flagged comments. The response table stores the rating, feedback and route, not the respondent's name or email.

Repository: [power-platform-event-survey-demo](https://github.com/MRodriguez2010/power-platform-event-survey-demo)

## Conditional mail merge (HRO)

**Stakeholders.** The HRO team (owners and senders) and employees (recipients). The source data included protected health information, so data handling was the main requirement.

**Problem.** Standard mail merge sends the same message and attachments to every row. HRO requests often had exceptions: different wording by category, an attachment for only some recipients, and people who had already been contacted.

**Flow.** For each recipient row, decide a status (checks run in this order), then send only rows that are `ReadyToSend`:

| Status | Rule | Action |
|---|---|---|
| `DuplicateNeedsReview` | Same recipient appears more than once in the list | Hold for review |
| `SuppressedAlreadyContacted` | Contact log shows a prior send | Skip |
| `HoldUncertainOutcome` | Last send attempt didn't confirm | Hold for a person to check, rather than risk a duplicate |
| `MissingAttachment` | Row requires an attachment that isn't present | Block |
| `ReadyToSend` | None of the above | Send, with or without attachment, then log |

After a send, the contact log records the state and increments the contact count, so the next run leaves that person out. A failed or timed-out send is logged as uncertain instead of being retried blindly.

**Why not a third-party tool.** The organization already licensed Power Automate and OneDrive. Building the merge there avoided new spend and kept protected health information inside the tenant.

**Runs** can be started manually for one-off requests or scheduled for recurring ones.

Repository: [power-platform-conditional-mail-demo](https://github.com/MRodriguez2010/power-platform-conditional-mail-demo)
