---
name: n8n-financial-workflow-builder
description: Build n8n workflows from financial process documents, diagrams, or descriptions. Use this skill when the user asks to build, create, or model a financial process as an n8n workflow — such as ETF creation/redemption models, cash flow processes, trade settlement flows, fund operations, or any multi-party financial lifecycle. Triggers on phrases like "build a workflow from this", "model this process in n8n", "create an n8n workflow for this financial process", or when a PDF/diagram of a financial process is provided.
---

# n8n Financial Process Workflow Builder

Builds n8n workflows that model financial processes from documents, PDFs, or diagrams.

---

## What This Skill Does

Reads a financial process document or diagram and automatically:
1. Identifies all parties involved
2. Maps every step in the process
3. Identifies what is being exchanged (cash, assets, messages, instructions)
4. Identifies timing (same day, T+1, T+2 etc.)
5. Builds and deploys a complete n8n workflow via the API

---

## Step 1 — Read and Understand the Document

When given a PDF or diagram, extract:

### Parties
List every entity involved. Common ones in financial workflows:
- Market Makers (regulated broker-dealer side AND offshore crypto side)
- Authorized Participant (AP)
- ETF Issuer / Fund Manager
- Transfer Agent & Cash Custodian
- Bitcoin / Asset Custodian
- Prime Broker
- Listing Exchange / Spot Exchanges
- Clearing House / DTC

### Steps
Number every step in order. For each step identify:
- **Who** is acting (the party)
- **What** they are doing (the action)
- **What moves** — is it an asset (cash, Bitcoin, shares) or just a message/instruction?
- **When** — which day/time (T, T+1, T+2 etc.)
- **Where** it goes (destination party or venue)

### Flow Types
- `Asset Movement` — actual money, Bitcoin, shares physically moving
- `Messaging` — instructions, approvals, confirmations (no asset moves)
- `Asset Movement + Messaging` — both happen together

---

## Step 2 — Design the Workflow Structure

### Node types to use:

| Purpose | n8n Node |
|---|---|
| Start the workflow | `n8n-nodes-base.manualTrigger` |
| Each process step | `n8n-nodes-base.set` (typeVersion 3.4) |
| Approval / decision point | `n8n-nodes-base.if` (typeVersion 2) |
| Day separator (T to T+1 etc.) | `n8n-nodes-base.noOp` (typeVersion 1) |
| Labels and context | `n8n-nodes-base.stickyNote` (typeVersion 1) |
| End / completion | `n8n-nodes-base.set` (typeVersion 3.4) |

### Layout rules:
- Arrange nodes **left to right** following the process order
- Use **y position** to separate time periods (T = y:280, T+1 = y:580 or continue right)
- Space nodes **220px apart** on the x axis
- Put sticky notes **above** the relevant section (lower y value)
- Put rejected/error branches **below** the main flow (higher y value)

### Positions guide:
- Trigger: [0, 400]
- Steps continue at +220 on x axis
- Day separator (noOp): place between last T-day step and first T+1 step
- Sticky note headers: y = 160-200 (above the nodes)
- Rejected branch: y = 540-600

---

## Step 3 — Build the Workflow JSON

### Set node format (each process step):
```json
{
  "parameters": {
    "assignments": {
      "assignments": [
        {"id": "unique-id", "name": "step", "value": "1", "type": "string"},
        {"id": "unique-id", "name": "timing", "value": "T", "type": "string"},
        {"id": "unique-id", "name": "party", "value": "Name of party acting", "type": "string"},
        {"id": "unique-id", "name": "action", "value": "Full description of what they do", "type": "string"},
        {"id": "unique-id", "name": "flowType", "value": "Asset Movement OR Messaging", "type": "string"},
        {"id": "unique-id", "name": "status", "value": "SNAKE_CASE_STATUS_CODE", "type": "string"}
      ]
    },
    "includeOtherFields": true,
    "options": {}
  },
  "id": "step-node-001",
  "name": "Step 1 [T] - Party: Action Description",
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4,
  "position": [220, 400]
}
```

Note: The **first** Set node after the trigger does NOT need `"includeOtherFields": true`. All subsequent ones do.

### IF node format (approval gates):
```json
{
  "parameters": {
    "conditions": {
      "options": {"caseSensitive": true, "leftValue": "", "typeValidation": "strict"},
      "conditions": [
        {
          "id": "cond-001",
          "leftValue": "={{ $json.approved }}",
          "rightValue": "true",
          "operator": {"type": "string", "operation": "equals"}
        }
      ],
      "combinator": "and"
    },
    "options": {}
  },
  "id": "if-node-001",
  "name": "Order Approved?",
  "type": "n8n-nodes-base.if",
  "typeVersion": 2,
  "position": [660, 400]
}
```

IF node outputs: index 0 = true/yes branch, index 1 = false/no branch

### Sticky note format:
```json
{
  "parameters": {
    "content": "## Title\n\nContent here",
    "height": 180,
    "width": 500,
    "color": 3
  },
  "id": "sticky-note-001",
  "name": "Section Label",
  "type": "n8n-nodes-base.stickyNote",
  "typeVersion": 1,
  "position": [-20, 160]
}
```

Colors: 1=grey, 2=brown, 3=green, 4=blue, 5=purple, 6=red, 7=orange

### Connections format:
```json
"connections": {
  "NodeName": {
    "main": [
      [{"node": "NextNodeName", "type": "main", "index": 0}]
    ]
  },
  "IF Node Name": {
    "main": [
      [{"node": "TrueBranchNode", "type": "main", "index": 0}],
      [{"node": "FalseBranchNode", "type": "main", "index": 0}]
    ]
  }
}
```

---

## Step 4 — Deploy to n8n

### n8n API details:
- **URL**: http://localhost:5678
- **API endpoint**: POST http://localhost:5678/api/v1/workflows
- **Auth header**: X-N8N-API-KEY

### PowerShell deployment:
```powershell
$headers = @{
    "X-N8N-API-KEY" = "<api-key>"
    "Content-Type" = "application/json"
}
$body = Get-Content -Raw -Path "$env:TEMP\workflow.json" -Encoding utf8
$response = Invoke-RestMethod -Uri "http://localhost:5678/api/v1/workflows" -Method POST -Headers $headers -Body $body -ContentType "application/json"
Write-Output "Workflow ID: $($response.id)"
Write-Output "URL: http://localhost:5678/workflow/$($response.id)"
```

Always write the JSON to a temp file first, then POST — avoids escaping issues.

---

## Step 5 — What to Include in Every Workflow

### Always include:
- [ ] Manual trigger node to start the workflow
- [ ] One Set node per process step (named clearly with step number and timing)
- [ ] IF node for every approval or decision point
- [ ] noOp node as a day separator between T and T+1 (or T+1 and T+2)
- [ ] Sticky notes labelling each time period
- [ ] Sticky note with parties legend
- [ ] Completion node at the end
- [ ] Rejected/failed branch off every approval IF node

### Data each step node must carry:
- `step` — step number
- `timing` — T, T+1, T+2 etc.
- `party` — who is acting
- `action` — full plain English description of the action
- `flowType` — Asset Movement / Messaging / Asset Movement + Messaging
- `status` — SNAKE_CASE status code (e.g. ORDER_PLACED, CASH_DELIVERED)

---

## Common Financial Process Patterns

### ETF Creation/Redemption (Cash Model)
- T-day: Order placed → Approved → Asset purchased → Custodian instructed → Trade agreed
- T+1: Cash delivered → Asset delivered → Shares surrendered → Cash released → Position unwound
- Key gate: Issuer approval IF node
- Key parties: Market Maker (BD + crypto), AP, ETF Issuer, Custodian, Transfer Agent, Prime Broker

### ETF Creation/Redemption (In-Kind Model)
- Similar but assets (ETF basket securities) move instead of cash
- No offshore entity needed — regulated broker-dealer handles directly

### Trade Settlement (DVP — Delivery vs Payment)
- Buyer sends payment simultaneously as seller delivers asset
- Key: simultaneous exchange, single settlement day

### Fund NAV Calculation
- End of day: collect prices → calculate NAV → publish → notify APs
- Scheduled trigger, multiple data source HTTP Request nodes

---

## Example — Completed Cash Redemption Workflow

Reference: workflow ID `8ucDfU0J47x75AQN` on http://localhost:5678
This is the BlackRock iShares Bitcoin ETF Cash Redemption Model built from the PDF.

It contains:
- 15 functional nodes (trigger, 10 step nodes, 1 IF, 1 noOp, 1 complete, 1 rejected)
- 3 sticky notes (T-day header, T+1 header, parties legend)
- Full T and T+1 flow across all 8 parties

Use this as a reference template for similar workflows.

---

## Checklist Before Deploying

- [ ] Every step from the document has a node
- [ ] Node names include step number and timing e.g. "Step 3 [T] - ..."
- [ ] includeOtherFields: true on all Set nodes except the first
- [ ] All connection names match node names exactly (case sensitive)
- [ ] IF nodes have both true and false branches connected
- [ ] Sticky notes are positioned above the relevant nodes (lower y value)
- [ ] Workflow has a clear start (trigger) and end (completion node)
- [ ] JSON is valid before POSTing
