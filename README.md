# n8n-financial-workflow-builder# n8n Financial Workflow Builder — Claude Skill

A Claude Code skill that reads any financial process document (PDF, diagram, or description) and automatically builds and deploys a fully structured n8n workflow.

---

## What It Does

Give it a financial process document and it will:
- Identify all parties involved
- Map every step in the process
- Determine what moves (cash, Bitcoin, ETF shares) vs what is just a message/instruction
- Build a complete n8n workflow with proper nodes, connections, and labels
- Deploy it directly to your n8n instance via the API

---

## Requirements

- [Claude Code](https://claude.ai/code) installed
- [n8n](https://n8n.io) running locally or on a server
- An n8n API key (Settings → API → Create API Key)

> **Also recommended:** Install the n8n skills pack for best results. It teaches Claude correct n8n expression syntax, node configuration, and workflow validation:
> ```
> /plugin install czlonkowski/n8n-skills
> ```

---

## Installation

### Option 1 — Plugin install (recommended)
```
/plugin install barnet-c/n8n-financial-workflow-builder
```

### Option 2 — Manual
1. Copy `skill.md` into your Claude skills folder:
```
~/.claude/skills/n8n-financial-workflow-builder/skill.md
```
2. Restart Claude Code

---

## How To Use

1. Open Claude Code or VS code chat
2. Drop in a PDF or describe a financial process
3. Type:
```
build a workflow from this
```
4. Claude will read the document, build the workflow JSON, and deploy it to your n8n
5. You get back a direct link to open it

---

## What the Output Looks Like in n8n

Each workflow includes:
- One node per step, clearly labelled with step number and timing (T, T+1 etc.)
- Approval gates (IF nodes) for any decision points
- Day separators between Trade Date and Settlement Date
- Sticky notes labelling each section and listing all parties
- A completion node and a rejected branch

---

## n8n API Setup

When Claude asks for your n8n details, provide:
- **URL**: your n8n instance URL (e.g. `http://localhost:5678`)
- **API Key**: from n8n → Settings → API → Create API Key

These are stored in `~/.claude/settings.json` and used automatically for all future workflows.

---

