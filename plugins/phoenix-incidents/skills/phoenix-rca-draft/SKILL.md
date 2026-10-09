---
name: phoenix-rca-draft
description: Draft a root cause analysis (RCA) for a Phoenix Incident Management incident before the RCA meeting. Use when the user asks to draft, prepare, pre-fill or summarize an RCA, postmortem or incident review for a Jira incident or RCA created by Phoenix Incident Management. Reads every Phoenix field and issue property in Jira, then gathers evidence from the team's own tools (logs, monitoring, chat, deploys).
---

# Draft a Phoenix RCA

Phoenix Incident Management keeps incident data in three places in Jira. Default Jira reads miss two of them. Follow these steps in order.

## 1. Find the two issues

This skill needs the Atlassian MCP server. If it is not connected, stop. Tell the user to connect it and give it access to the Jira site where Phoenix runs. Other Jira tools may not return issue properties, and the draft would be missing the timeline.

- The **Incident** is a normal issue of type `Incident`.
- The **RCA** is a sub-task of type `RCA` under the incident. Its summary starts with "RCA for".
- The incident and RCA are linked only as parent and sub-task, not by an issue link.
- The user may give either key. Read the one you have, then read the other through `parent` or `subtasks`.
- If the Atlassian MCP cannot reach the site, say so. Ask the user to add that site to the connector. Do not guess the data.

## 2. Read everything, not the defaults

Read **both** issues with all fields, all issue properties and field names:

| Connector | Settings to pass on get issue |
|---|---|
| Atlassian MCP v1 (tools like `getJiraIssue` with `fields`) | `fields: ["*all"]`, `properties: ["*all"]`, `expand: "names"` |
| Atlassian MCP v2 (tools with a `view` option) | `view: "full"`, `properties: ["*all"]` |

Without these, you will miss the Five Whys, severity, start and end times, the timeline and every RCA text field.

Custom field IDs (`customfield_12345`) differ on every site. Always match fields by **name** from the `names` map, never by ID.

Read the comments on both issues too. Some connectors only return a comment count by default. Use the comments tool if so.

## 3. What the data means

### Incident fields (match by name)
- `Incident Severity`, `Primary Incident Source`, `Impacted Service(s)`
- `Incident Start`, `Incident End`, `Incident Duration` (minutes)
- `Business Impact`, `Functional Impact`, `Root Causes`, `Executive Summary`
- `Response Channel` (the chat channel used during the incident), `Meeting Link`, `Paging Link`

### RCA fields (match by name)
- `Five Whys` (see below)
- `Executive Summary`, `Impact to Customers`, `Summary of Resolution`, `Error Message`, `Additional Notes/Observations`
- `Meeting Participants`, `Reviewers`
- RCA status moves through: Data Gathering → Analysis Meeting → Under Review → Finalized.

### Issue properties on the incident
The incident's issue properties hold the Phoenix timeline, the time it entered each status, and the alert that opened it. Read them all. Some timeline entries are made by Phoenix, such as start, end and status changes. Others were added by people, including chat messages pinned to the timeline. Ignore properties that only describe screen layout.

### Linked issues on the incident
Phoenix links two kinds of issues to the incident. Read each linked issue's summary, status, assignee and due date.
- **Action items**: link "has action item" / "is an action item of". Work the team agreed to do because of this incident. They may be added before or after the meeting.
- **Contributing factors**: link "has contributing factors" / "is a contributing factor of". Changes that helped cause the incident, such as deploys or change tickets.

### Five Whys field
It holds JSON text. Read every question and answer in it.
- A question can split into several answers, each with its own whys. Number them like an outline: 1, 2, 3. The splits of question 2 are 2.a and 2.b. The whys in split 2.a are 2.a.1, 2.a.2.
- It may hold next questions Phoenix suggested. These have no answer yet. Some are flagged as a likely root cause. Treat a flag as a hint, not a fact.
- Older RCAs use version 2, a straight list with no splits: `{"version": 2, "entries": [{"question": "...", "answer": "...", "isBranched": false}]}`. Number it 1, 2, 3. Ignore `isBranched`. Phoenix does not use it.
- Some connectors escape the brackets in this text (`\[`, `\]`). Remove the backslashes before reading it.

## 4. Gather evidence from the team's tools

Check which tools you have. For each kind below, use it if connected. If not, tell the user what it would add and ask if they want to connect one before you continue. Do not stop if they say no.

| Kind | Examples | What to look for |
|---|---|---|
| Chat | Slack, Microsoft Teams | Messages in the `Response Channel` during the incident: first report, who did what, the fix |
| Logs and errors | Sentry, Datadog Logs, CloudWatch, Splunk | First error, error spike, error message text |
| Monitoring | Datadog, New Relic, Grafana, Prometheus | When metrics went bad and came back |
| Deploys and changes | GitHub, GitLab, CI, AWS, change tickets | Deploys or config changes shortly before `Incident Start` |
| Paging | Phoenix Alerts, PagerDuty, Opsgenie, VictorOps (Splunk On-Call) | When the alert fired and who acknowledged it |

The examples are not a full list. Use any connected tool that fits a kind, even one not named here. If you are not sure what kind a tool is, ask the user.

Search from 2 hours before `Incident Start` to 1 hour after `Incident End`. Times in Jira carry their own offset. Convert everything to one time zone and say which.

## 5. Write the draft

Use these sections.

1. **Summary**: what broke, for how long, who was affected. Two or three sentences.
2. **Impact**: customers, services, severity, duration.
3. **Timeline**: one table, oldest first. Columns: time, what happened, source. Merge the Phoenix timeline, linked contributing factors and what you found. Mark each row's source (`Phoenix timeline`, `Slack`, `Datadog`, ...).
4. **Five Whys so far**: the current tree with its numbers. List Phoenix's suggested next questions under the line they follow, and mark the flagged ones. If it is empty, say the team will do it in the meeting.
5. **Leads for the Five Whys**: the evidence that will help the team answer the first "why". Facts only. Do not answer the whys or name a root cause.
6. **Resolution**: what fixed it.
7. **Gaps and questions for the meeting**: missing data, conflicting times, empty fields, open Five Whys lines.
8. **Action items**: first list the linked action items with key, summary, status and owner. Say which root cause each one covers. Then suggest new ones only for root causes the team already reached in the Five Whys and no linked item covers. Never repeat a linked item. Leave this section out if there is nothing to list.

### How to write
- Plain English. Short sentences. Professional tone.
- Use the correct technical terms: service names, error messages, metrics. Explain an acronym the first time it appears.
- Avoid jargon that a plain word can replace.
- Blameless: describe what happened and what allowed it, not who is at fault. Name people only for who did what in the timeline.

### Five Whys: live by default
Most teams should do the Five Whys live in the RCA meeting. Do not write the whys for them unless the user asks.

If the user asks for a Five Whys draft, add a section **Suggested Five Whys**. Label it "Suggestion for the meeting to confirm or change." Base each answer on evidence and name the source. You may then add likely root causes and action items tied to them. Never write it into the Five Whys field. People enter it in Phoenix.

Rules:
- Every fact needs a source. If you infer something, say "likely" and why.
- Never invent times, names or numbers. Leave a gap and list it in section 7.
- Empty Phoenix fields are gaps to list, not things to fill with guesses.

## 6. Hand it over

Show the draft to the user in the chat. It is for them to read before the meeting. Do not post it to Jira.

Then offer to fill the empty RCA text fields. Do this only after they say yes. These are safe to edit:
- `Executive Summary`, `Impact to Customers`, `Summary of Resolution`, `Error Message`, `Additional Notes/Observations`

Rules for filling:
- Only fill a field that is empty. Never overwrite what a person wrote. If a field has text, leave it alone. Your version is already in the draft.
- Only while the RCA status is Data Gathering or Analysis Meeting. Never touch an RCA that is Under Review or Finalized.
- Start each field with the line "AI draft. Review before the RCA is finalized."
- List which fields you filled.

Do **not** edit these yourself:
- any issue property, including the timeline
- the `Five Whys` field
- the RCA or incident status
- comments. Do not post the draft or anything else as a comment.
- any incident field
- issue links. Do not create action items or link issues. Suggest them in the draft instead.

A hand-written value in these can break the Phoenix screens. People add the draft's timeline rows and whys through the Phoenix screens during the meeting.
