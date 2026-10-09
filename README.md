# Phoenix AI

AI skills for Phoenix Incident Management in Jira.

## Skills

### phoenix-rca-draft

Drafts a root cause analysis (RCA) before your RCA meeting.

It reads the incident and RCA in Jira, including the Phoenix timeline, Five Whys and status dates. It also checks the tools you have connected, such as Slack, Sentry, Datadog or GitHub. Any chat, logs, monitoring, deploy or paging tool works. Then it writes a draft with a summary, impact, timeline, root causes, gaps and action items.

It asks before it changes anything. It only fills empty RCA text fields, and it never edits the timeline, the Five Whys or the status.

Try: "Draft the RCA for INC-123."

## Before you start

Connect your AI tool to Jira with the Atlassian MCP server (Atlassian Rovo MCP). The skill needs it and does not work with other Jira tools. Make sure it has access to the Jira site where Phoenix runs.

For a better draft, also connect your chat, logs, monitoring and code tools.

## Install

The skill is one folder: [`plugins/phoenix-incidents/skills/phoenix-rca-draft`](plugins/phoenix-incidents/skills/phoenix-rca-draft). It uses the open [Agent Skills](https://agentskills.io) format, so the same folder works in every tool below.

**Claude Code**

```
/plugin marketplace add Phoenix-Incidents/phoenix-ai
/plugin install phoenix-incidents@phoenix-ai
```

**Claude (web and desktop)**

Download the `phoenix-rca-draft` folder as a zip. Upload it in Claude's settings under Skills.

**OpenAI Codex and the ChatGPT desktop app**

Copy the `phoenix-rca-draft` folder into `~/.agents/skills/`.

**Gemini CLI**

```
git clone https://github.com/Phoenix-Incidents/phoenix-ai.git
gemini skills install ./phoenix-ai/plugins/phoenix-incidents/skills/phoenix-rca-draft
```

Or copy the folder into `~/.gemini/skills/`.

**Cursor**

Copy the `phoenix-rca-draft` folder into `~/.cursor/skills/`.

## Not supported yet

ChatGPT on the web and mobile, and the Gemini app, cannot load skills from a folder.
