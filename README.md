# say less _

**Open-source Claude Skills for your inbox — no dashboards, no forms, just conversation.**

> "Clean up my inbox." → done.
> "Show me what I spent on DoorDash last month." → CSV exported.

Say Less is a free, open-source collection of [Claude Skills](https://www.anthropic.com/claude) that connect to your Gmail and let you manage your inbox through plain English. No new apps to learn. No settings to configure. Just talk to Claude.

---

## Skills

### 🗑 Inbox Detox

Scans your inbox for the biggest email offenders, groups them by sender, and lets you unsubscribe and bulk-delete entire categories in a single conversation.

- Surfaces top senders by volume (Wayfair, LinkedIn, Medium, etc.)
- Handles unsubscribing via Gmail's native button or in-email links
- Bulk-deletes hundreds of emails per sender
- Always confirms before deleting — never touches transactional email without asking
- Works in batches of 5–10 senders so you stay in control

**Example:**
```
you:        clean up my inbox
say less:   Found 142 senders across ~2,400 emails. Top offender: Groupon — 340 emails.
            Want to go category by category, or just the worst offenders?

you:        kill all the marketing stuff, keep my newsletters
say less:   Done. 1,100 emails from 58 senders cleared. Your newsletters are untouched.
```

---

### 🧾 Receipt Scanner

Searches Gmail for purchase receipts and order confirmations, parses structured data (merchant, amount, payment method, category), and exports a clean downloadable CSV.

- Covers DoorDash, Lyft, Anthropic, Google Play, PayPal, Amazon, and dozens more
- Auto-categorizes spending: Food Delivery, Transportation, Subscriptions, Utilities, etc.
- Exports a CSV with totals and a summary breakdown by category
- Deduplicates receipts across multiple confirmation emails for the same order

**Example:**
```
you:        what did I spend in January?
say less:   Scanned 847 emails. Found 63 receipts.
            Total spend: $1,842.17
            Top categories: Food Delivery ($312), Subscriptions ($97), Transportation ($84)

            Exporting CSV now...
```

---

## Getting Started

### Prerequisites

- **Claude Desktop** (or Claude.ai with Skills support)
- **Gmail integration** enabled in Claude — Settings → Integrations → Gmail
- **Claude in Chrome** (recommended for Inbox Detox — enables one-click unsubscribe)

### Install a Skill

1. Open **Claude Desktop**
2. Go to **Settings → Skills**
3. Click **Add Skill** (or **Import Skill**)
4. Select the skill folder containing `SKILL.md`
   e.g. `sayless_skills/inbox-detox` or `sayless_skills/receipt-scanner`
5. Confirm and enable

Done. Open a new conversation and just start talking.

---

## Skills Directory

```
sayless_skills/
├── inbox-detox/       # Bulk unsubscribe + delete marketing email
│   └── SKILL.md
└── receipt-scanner/   # Parse Gmail receipts → CSV export
    └── SKILL.md
```

Each skill is a single `SKILL.md` file — a structured prompt that tells Claude how to behave, what tools to use, and how to handle edge cases. No code to run, no servers to deploy.

---

## How It Works

Claude Skills extend Claude with domain-specific instructions. Say Less skills use Claude's **Gmail connector** to read and search your email — no third-party services, no API keys, nothing leaves your Claude session.

```
you → Claude → Gmail connector → your inbox
```

Claude reads the `SKILL.md` instructions and follows the workflow: scan, surface, confirm, act. You stay in control of every deletion.

---

## Safety

- **Always confirms before deleting.** Claude states the sender and approximate count, then waits for explicit approval.
- **Protects transactional email.** Receipts, order confirmations, and password resets are flagged before any action is taken.
- **Metadata-first scanning.** Inbox Detox reads only sender/subject headers during the initial scan — it never reads full email bodies unless needed to find an unsubscribe link.
- **One message body read per sender, maximum.** If Gmail's native unsubscribe button isn't visible, Claude reads the single most recent email from that sender. Nothing more.

---

## Contributing

Skills are just Markdown files with a structured front-matter block and natural language instructions. If you have an idea for a new inbox-related skill, open a PR with a new folder under `sayless_skills/`.

Skill structure:
```
sayless_skills/<skill-name>/
├── SKILL.md       # Required — skill definition and instructions
├── README.md      # Optional — human-readable overview
├── templates/     # Optional — reusable prompt fragments
└── examples/      # Optional — example inputs/outputs
```

---

## License

MIT — free to use, modify, and distribute.

---

Built by [Spenser Marshall](https://github.com/spenserkm) · Powered by [Claude](https://www.anthropic.com/claude)
