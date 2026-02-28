# Sayless Skills

This directory contains installable Claude skills for the Sayless project.

## Install on Claude Desktop

Claude Desktop skill support can vary by version. Use one of the following methods depending on what your app shows.

### Option A: Install from folder (recommended in this repo)

1. Open Claude Desktop.
2. Go to **Settings** -> **Skills**.
3. Click **Add Skill** (or **Import Skill**).
4. Select the skill folder that contains `SKILL.md` (for example, `sayless_skills/receipt-scanner`).
5. Confirm installation and enable the skill.

### Option B: Install from packaged `.skill` file

1. Open Claude Desktop.
2. Go to **Settings** -> **Skills**.
3. Choose **Import** and select a `.skill` package file.
4. Confirm installation and enable the skill.

## Available Skills

### `receipt-scanner`

- **Path:** `sayless_skills/receipt-scanner`
- **Purpose:** Searches Gmail for receipts and order confirmations, extracts purchase details, and exports purchase history to CSV.
- **Primary use cases:** spending audits, purchase history exports, order/receipt summaries.
- **Prerequisite:** Gmail connector enabled in Claude.

## Skill Structure Convention

Each skill should follow this layout:

```text
<skill-name>/
├── SKILL.md
├── templates/    (optional)
├── examples/     (optional)
└── README.md
```
