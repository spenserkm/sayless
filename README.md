<div align="center">

# say less _

### Open-source Claude Skills for people with better things to do than manage their inbox.

</div>

<div align="center">

[![License: MIT](https://img.shields.io/badge/license-MIT-green)](https://opensource.org/licenses/MIT)
[![Skills](https://img.shields.io/badge/skills-2-c8ff00?labelColor=111)](./sayless_skills)
[![Claude](https://img.shields.io/badge/powered%20by-Claude-blueviolet)](https://www.anthropic.com/claude)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)](CONTRIBUTING.md)

</div>

Say Less is a free, open-source collection of [Claude Skills](https://www.anthropic.com/claude) that connect to your Gmail and let you manage your inbox through plain English. No new apps to learn. No API keys. No dashboards. Just talk.

```
you:       clean up my inbox
say less:  Found 142 senders across ~2,400 emails. Top offender: Groupon — 340 emails.
           Want to go category by category, or just the worst offenders?

you:       kill all the marketing stuff, keep my newsletters
say less:  Done. 1,100 emails from 58 senders cleared. Your newsletters are untouched.
```

**Install:**
1. Clone or download this repo
2. Open **Claude Desktop → Settings → Skills → Add Skill**
3. Point it at a skill folder (e.g. `sayless_skills/inbox-detox`)
4. Talk to Claude

---

## Why Say Less?

Most inbox tools want you to log into a new product, learn a new UI, and trust a third-party service with your email. Say Less skips all of that.

Use Say Less when you want to:

- **Reach inbox zero without a new app.** Everything runs inside Claude. No new accounts, no OAuth flows to a stranger's server.
- **Clean email through conversation.** "Unsubscribe from everything retail" or "delete all LinkedIn notifications" — natural language, immediate action.
- **Audit your spending from receipts.** Claude scans your inbox, parses structured purchase data from dozens of merchants, and drops a CSV on your desktop.
- **Stay in control.** Say Less always tells you what it's about to do and waits for a yes before deleting anything.
- **Extend it yourself.** Each skill is a single Markdown file. Write one in 20 minutes with zero code.

> [!NOTE]
> Say Less requires the **Gmail integration** to be enabled in Claude. Go to **Settings → Integrations → Gmail** and connect your account before installing any skill.

---

## Skills

### 🗑 Inbox Detox

Bulk unsubscribe and delete marketing email through conversation. No more hunting through individual emails for tiny unsubscribe links.

**What it does:**
- Scans your Promotions, Social, Spam, and Updates tabs
- Groups results by sender and sorts by volume — biggest offenders first
- Shows 5–10 senders at a time; you decide keep / unsubscribe + delete / delete only / skip
- Executes unsubscribes via Gmail's native button, then bulk-deletes all matching emails
- Falls back to copy-paste Gmail search queries if Chrome isn't available

```
you:       my inbox is a disaster
say less:  Working with you@gmail.com. Start with promos and newsletters,
           or go broader and hit social notifications and spam too?

you:       promos first
say less:  Top senders in Promotions:
           1. Wayfair <deals@wayfair.com> — 47 emails
           2. LinkedIn <messages-noreply@linkedin.com> — 38 emails
           3. Medium <noreply@medium.com> — 31 emails

           Keep, unsubscribe + delete, delete only, or skip?

you:       unsubscribe and delete Wayfair. keep medium. skip LinkedIn.
say less:  About to unsubscribe + delete ~47 emails from Wayfair. Go ahead?

you:       yes
say less:  Done. Unsubscribed from Wayfair + deleted 47 emails.
           Continue with the next batch, or switch to another category?
```

---

### 🧾 Receipt Scanner

Searches Gmail for purchase receipts and order confirmations, extracts structured data, and exports a downloadable CSV. Useful for spending audits, expense reports, or just figuring out how much you actually spent on DoorDash.

**What it does:**
- Runs multiple search queries to maximize receipt coverage
- Parses merchant, product, price, payment method, and order number per email
- Handles merchant-specific formats: DoorDash, Lyft, Anthropic, Google Play, PayPal, Amazon, and more
- Auto-categorizes: Food Delivery, Transportation, Subscriptions, Utilities, Healthcare, and others
- Exports a sorted CSV with a total spend row and a category breakdown summary

```
you:       what did I spend last month?
say less:  Scanning receipts from Jan 1 – Jan 31...
           Found 63 purchases across 847 emails.

           Total spend: $1,842.17

           By category:
           Food Delivery    $312.44   (18 orders)
           Subscriptions    $97.00    (6 services)
           Transportation   $84.20    (4 trips)
           Utilities        $210.00   (3 bills)
           ...

           Exporting purchase_history.csv now.
```

---

## How It Works

Claude Skills extend Claude with domain-specific instructions loaded at conversation start. Say Less skills use Claude's built-in **Gmail connector** — no third-party service ever touches your email.

```
you ──► Claude (+ SKILL.md instructions)
             │
             ▼
        Gmail connector
             │
             ▼
        your inbox
```

Each `SKILL.md` file tells Claude: what to trigger on, what tools to call, how to parse results, and what safety rules to follow. There's no server, no runtime, and no deployment — it's just a structured prompt.

---

## Safety

Say Less is intentionally conservative about destructive actions.

- **Confirm before delete.** Claude always states the sender name and approximate count, then waits for an explicit yes. No exceptions.
- **Transactional email protection.** Anything that looks like a receipt, order confirmation, shipping update, bank statement, or password reset gets flagged before any action is suggested.
- **Metadata-only scanning.** Inbox Detox reads only `From` and `Subject` headers during the initial scan. It never reads full email bodies unless it needs to find an unsubscribe link.
- **One body read per sender, maximum.** If Gmail's native unsubscribe button isn't available, Claude reads the single most recent email from that sender — nothing more.
- **No speculative pagination.** Searches stop at the first page of results unless you explicitly ask for a deeper scan.

---

## Skills Directory

```
sayless_skills/
├── inbox-detox/       🗑  Bulk unsubscribe + delete marketing email
│   └── SKILL.md
└── receipt-scanner/   🧾  Parse Gmail receipts → CSV export
    └── SKILL.md
```

Skills follow a simple convention:

```
sayless_skills/<skill-name>/
├── SKILL.md       # Required — triggers, instructions, workflow, safety rules
├── README.md      # Optional — human-readable summary
├── templates/     # Optional — reusable prompt fragments
└── examples/      # Optional — example inputs/outputs
```

---

## Contributing

Want to add a new skill? Skills are just Markdown — no code required.

A `SKILL.md` has two parts: a YAML front-matter block (name + description/trigger) and a natural language workflow that Claude follows. If you can describe a useful inbox task in plain English, you can write a skill.

Open a PR with a new folder under `sayless_skills/`. Check the existing skills for the expected format.

> [!TIP]
> Good skill ideas: calendar cleanup, attachment organizer, contact deduplication, newsletter digest, unread triage. If it touches Gmail and you'd do it yourself in 30 minutes, Claude can probably do it in 30 seconds.

---

## Additional Resources

- [Claude Skills documentation](https://www.anthropic.com/claude) — how Skills work in Claude Desktop
- [Gmail integration guide](https://www.anthropic.com/claude) — how to connect Gmail to Claude
- [Claude in Chrome](https://www.anthropic.com/claude) — enables one-click unsubscribe and bulk delete in the Gmail UI

---

<div align="center">

MIT License · Built by [Spenser Marshall](https://github.com/spenserkm) · Powered by [Claude](https://www.anthropic.com/claude)

</div>
