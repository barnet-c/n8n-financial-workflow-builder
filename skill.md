---
name: etf-prospectus-reader
description: Reads any fund or ETF prospectus (S-1, S-1/A, offering document, or similar legal filing) and extracts the creation and redemption mechanics into a clean, structured process flow document. Use this skill when the user provides a prospectus PDF, S-1 filing, or legal document and wants to understand or model the creation/redemption process. Triggers on phrases like "read this prospectus", "extract the workflow from this S-1", "make a process doc from this filing", "summarise the creation/redemption mechanics". Output is a structured markdown document suitable for direct input into the n8n-financial-workflow-builder skill.
---

# ETF / Fund Prospectus Reader — Process Flow Extractor

Reads any fund or ETF prospectus and converts dense legal text into a clean, structured process flow document that can be used directly as input for the n8n-financial-workflow-builder skill.

Works with any product type: Bitcoin ETFs, equity ETFs, bond funds, commodity trusts, crypto funds, or any other pooled investment vehicle that has a creation/redemption mechanism.

---

## What This Skill Does

Takes a prospectus (S-1, S-1/A, fund document, or any legal filing) and outputs:
1. A **Product Summary** — what the fund holds, how it's structured
2. A **Parties List** — every entity involved and their role
3. A **Creation Flow** — numbered steps with timing, parties, actions, asset movements
4. A **Redemption Flow** — numbered steps with timing, parties, actions, asset movements
5. A **Special Mechanisms** section — anything structurally unique to this product

---

## Step 1 — Read and Locate the Relevant Sections

Prospectus documents are long. Scan the table of contents and focus only on:

### Sections to find and read:
- **"Creation and Redemption of Shares"** or **"Creation and Issuance"**
- **"Basket"** definitions — Creation Basket / Redemption Basket
- **"Authorized Participants"** section
- **"Custodian"** / **"Trustee"** / **"Administrator"** roles
- Any **financing, credit, or prepay** mechanisms
- **"Settlement"** timing sections
- Any **diagrams or flow descriptions** embedded in the document

### Sections to skip:
- Risk factors
- Tax sections
- Legal opinions
- Underwriting
- Financial statements
- Fee tables (unless a fee directly affects the flow)

---

## Step 2 — Extract All Parties

Read the document and list every entity that appears in the creation/redemption sections. Do not assume any parties — only list what the document states.

For each party record:
- A short name (used throughout the document)
- Their full legal name (as stated in the prospectus)
- Their role in the creation/redemption process

```
| Short Name | Full Legal Name | Role |
|---|---|---|
| [name] | [full legal name from doc] | [role as described in doc] |
```

Common party types to look for (names will vary by product):
- The entity that submits creation/redemption orders (often called "Authorized Participant")
- The entity that manages or sponsors the fund
- The entity that holds the underlying assets (custodian)
- The entity that holds cash
- The entity that executes trades
- The entity that issues or cancels shares (clearing)
- Any financing or credit provider
- Any administrator or transfer agent

---

## Step 3 — Map the Creation Flow

Extract every step from order submission through to share issuance.

For each step identify:
- **Step number** (sequential)
- **Timing** — which day: T (order day), T+1, T+2, or same-day
- **Acting party** — who initiates this step
- **Receiving party** — who receives it (if applicable)
- **Action** — plain English description of exactly what happens
- **What moves** — what asset or instruction is exchanged (cash, shares, the underlying asset, an instruction, or nothing)
- **Flow type** — one of:
  - `Asset Movement` — actual assets physically moving (money, securities, crypto)
  - `Messaging` — instructions, approvals, confirmations (nothing moves)
  - `Asset Movement + Messaging` — both happen together

### Creation mechanics to watch for:
- **Cash creation** — the AP delivers cash and the fund buys the underlying asset
- **In-kind creation** — the AP delivers the underlying asset directly
- **Partial cash** — a mix of cash and assets
- **Financing / Prepay / Trade Credit** — fund borrows something on T and repays on T+1
- **Approval gates** — does any party need to approve before the flow continues?

---

## Step 4 — Map the Redemption Flow

Same format as creation but in reverse — from share surrender through to asset delivery.

### Redemption mechanics to watch for:
- **In-kind redemption** — AP receives the underlying asset
- **Cash redemption** — fund sells the asset, AP receives cash
- **Who decides the type** — can the AP choose, or does the Sponsor decide?
- **Financing on redemption** — does the fund borrow on T to release assets before AP pays?

---

## Step 5 — Identify Special Mechanisms

Note anything that makes this fund structurally different from a standard creation/redemption model.

Ask these questions while reading:
- Is there any intraday financing or credit involved?
- Is there an offshore entity or a two-sided structure (regulated + unregulated)?
- Does the fund write options or hold derivatives?
- Is there a prepayment step before the main asset exchange?
- Is the basket composition unusual (mixed assets, synthetic exposure)?
- Are there any same-day settlement features that differ from the norm?

Describe each mechanism in plain English — what it is, why it exists, and how it changes the flow.

---

## Step 6 — Output the Structured Document

Produce a document in this exact format:

```markdown
# [Product Name] — Process Flow Document

## Product Summary
- **Fund Name:** [full name from prospectus]
- **Ticker:** [ticker if mentioned]
- **Underlying Asset:** [what the fund holds]
- **Creation Type:** [Cash / In-Kind / Either / At Sponsor's discretion]
- **Redemption Type:** [In-Kind / Cash / Either / At Sponsor's discretion]
- **Settlement:** [T+1 / T+2 / same day]
- **Special Mechanism:** [describe briefly, or "None"]

---

## Parties

| Short Name | Full Legal Name | Role |
|---|---|---|
| [short] | [full legal name] | [role] |

---

## Creation Flow

### T — Order Day

**Step 1 — [Acting Party]: [Action Title]**
- Party: [who acts]
- Action: [plain English description]
- What moves: [asset or "Nothing — instruction only"]
- Flow type: [Asset Movement / Messaging / Asset Movement + Messaging]

[continue for all T-day steps]

---

### T+1 — Settlement Day

[continue for all T+1 steps in same format]

---

## Redemption Flow

### T — Order Day

[same format as creation]

---

### T+1 — Settlement Day

[same format]

---

## Special Mechanisms

### [Mechanism Name]
[Plain English explanation — what it is, why it exists, how it affects the steps]

---

## Decision Points (Approval Gates)

- [Step N] — [Party] approves/rejects — if rejected: [what happens]

---

## Notes and Ambiguities

- [Anything the prospectus leaves vague or at Sponsor's discretion]
- [Any steps where exact timing or party is unclear]
```

---

## Output Instructions

- Use plain English throughout — no legal jargon
- Only include parties and steps that are actually stated in the prospectus — do not assume
- If the prospectus is ambiguous about a step, note it in the Ambiguities section rather than guessing
- If there are multiple creation/redemption models described (e.g. both cash and in-kind options), produce a separate flow section for each
- Save the document to a temp file using the Write tool — name it `<etf-name-slug>-process-flow.md` and save it to the system temp directory (use `$env:TEMP` on Windows or `/tmp` on Mac/Linux)
- After saving, tell the user the full file path
- After outputting the document, ask the user: **"Would you like me to now build this as an n8n workflow?"** — if yes, pass the document directly to the n8n-financial-workflow-builder skill
