# LedgerPro — Accounting Practice CRM & 5-Suite Automation Template

A demo-ready CRM for Indian accounting and tax practices, plus the matching
**n8n automation workflow** that runs the practice: invoicing with partner
approval, payment recovery across email / WhatsApp / AI voice calls, GST
compliance cycles, document collection, and an AI support assistant that
answers clients on web chat, WhatsApp and voice in **Hindi, English or
Hinglish**.

| File | What it is |
|---|---|
| `ledgerpro-crm.html` | The whole CRM in one HTML file — no build, no install, no backend |
| `Accounting Services - 5 Automation Suite (Template).json` | The n8n workflow — import and connect your accounts |

---

## 1. The CRM demo (`ledgerpro-crm.html`)

One self-contained file. Open it in any browser — that's the whole install.

```bash
# any of these work
start ledgerpro-crm.bat        # or just double-click the file
python -m http.server 8080     # then visit http://localhost:8080/ledgerpro-crm.html
```

Everything runs client-side: demo data is seeded in memory, your **Settings**
values persist in `localStorage` (keys are prefixed `ledgerpro_`), and the
support-desk AI is a rule-based stand-in that mirrors the real n8n agent's
behaviour — including Hindi/Hinglish detection and the same escalation rules.

### Modules

| Module | What it does |
|---|---|
| **Dashboard** | KPIs, receivables ageing, invoiced-vs-collected trend, deadlines, automation schedule, AI insights |
| **Clients** | Client master with GSTIN, plan, outstanding balance, GST and document status |
| **Invoices** | Auto GST maths, approval routing above the threshold, email + WhatsApp dispatch, payment tracking |
| **Payment Recovery** | Overdue ladder (gentle → firm → final → escalate) with per-stage email, WhatsApp and AI voice contact |
| **GST Compliance** | Monthly cycle: collect data → compute net payable → client approves → file |
| **Document Collection** | Monday chase for pending documents; after the reminder limit it escalates to a human call task |
| **Support Desk** | One AI assistant across web chat, WhatsApp and voice; unsure questions hand off to the team |
| **Outreach Log** | Every email, WhatsApp message and voice call sent on the firm's behalf — one searchable history |
| **Team Tasks** | Approvals, escalations and call-backs the automations hand to a person |
| **Automations** | Pause/resume each suite, run on demand, review run history |
| **Settings** | Firm profile, links, thresholds and every connection (see below) |

### Rules you can change in Settings → Rules & Thresholds

| Rule | Default | Controls |
|---|---|---|
| Partner approval above | ₹50,000 | Invoices above this wait for partner sign-off |
| Default GST rate | 18% | Pre-filled on new invoices |
| Default due days | 15 | Invoice due date |
| Gentle / Firm / Final cuts | 7 / 15 / 30 days overdue | Recovery escalation ladder |
| Document reminder limit | 3 | When chasing becomes "call the client" |

### Try these in the Support Desk

- `What is the due date for GSTR-3B?` → due date + the client's own return status
- `Mera GSTR-1 kab bharna hai?` → Hindi reply
- `Invoice INV-1003 ka status batao` → looks the invoice up (the mock tool the real agent uses)
- `Can you advise me on mutual fund investments?` → politely refuses and hands off to the team
- Anything it doesn't recognise → *"a team member will follow up within one working day"*

---

## 2. The n8n automation suite (`.json`)

Five automations in one importable workflow. Import order and connections below.

### Import

1. In n8n: **Workflows → ⋯ → Import from file**
2. Pick `Accounting Services - 5 Automation Suite (Template).json`
3. The workflow imports **inactive** — nothing sends until you connect accounts (Section 3)

### What's inside

| # | Automation | Trigger | Flow |
|---|---|---|---|
| 1 | **Invoicing** | Webhook `POST /accounting/new-invoice` | Calculate GST + totals → push to accounting software → **> ₹50,000?** partner approval email, else client invoice email → WhatsApp notification |
| 2 | **Payment recovery** | Schedule, daily 10:00 | Overdue invoices → switch by days overdue → gentle (7d) / firm (15d) / final notice (30d) / escalate (30d+) → each client stage sends **email → WhatsApp → AI voice call** |
| 3 | **GST compliance** | Schedule, monthly (1st, 09:00) | Per client: data received? → yes: compute net GST and send filing summary for approval; no: request sales/purchase registers (email + WhatsApp + voice) |
| 4 | **Document collection** | Schedule, Monday 10:00 | Pending documents → reminded 3+ times? → alert team to call, else send reminder (email + WhatsApp + voice) |
| 5 | **AI client support** | Chat trigger + WhatsApp webhook + voice webhook | One OpenAI agent with memory and an invoice-status tool → replies route back by channel |

> The import uses **mock data nodes** so every branch can be tested before
> anything real is connected. Replace each mock node with your real source
> (Zoho/Tally/QuickBooks export, Google Sheet, or database) as you activate
> each suite.

---

## 3. Setup & connections

Work through this once, top to bottom. The CRM's **Settings** page mirrors
these same connections — the values you enter there are what the demo uses in
its messages.

### 3.1 Credentials to create in n8n

**Credentials → Add credential**, one per row:

| Credential type | Used by | What you need |
|---|---|---|
| **Gmail OAuth2** | All invoice / reminder / GST / document emails | A Google account with the Gmail API enabled — n8n walks you through OAuth |
| **OpenAI** | The support agent's Chat Model node | API key from platform.openai.com |
| *(HTTP — no credential)* | WhatsApp + Sarvam voice nodes | Tokens pasted directly into the nodes (below) |

The imported file references a Gmail credential named `fitness stiudio` and an
OpenAI credential — **re-select your own** on every Gmail node and the Chat
Model node after import; n8n marks them with a warning icon until you do.

### 3.2 Replace every placeholder

Search the workflow for `YOUR_` and `yourfirm` and replace all of them:

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_PHONE_NUMBER_ID` | Every WhatsApp HTTP node | Meta WhatsApp Cloud API phone number ID |
| `YOUR_WHATSAPP_ACCESS_TOKEN` | Every WhatsApp HTTP node | Permanent access token from Meta Business settings |
| `YOUR_SARVAM_API_KEY` | Every voice-call HTTP node | Sarvam AI API key |
| `YOUR_ORG_ID` / `YOUR_WORKSPACE_ID` | Voice-call URLs | Your Sarvam org and workspace IDs |
| `YOUR_SARVAM_AGENT_APP_ID` / `YOUR_CONNECTION_ID` / `YOUR_AGENT_PHONE_NUMBER` | Voice-call bodies | Your Sarvam agent app config |
| `partner@yourfirm.example.com`, `team@yourfirm.example.com` | Internal escalation emails | Real partner / team inbox |
| `https://pay.yourfirm.example.com/pay` | Invoice + reminder messages | Your real payment link |
| `https://upload.yourfirm.example.com` | Document + GST data requests | Your real upload portal |
| `Your Firm Name` | Email subjects and signatures | Your firm name |

### 3.3 Mock data → real data

Each suite starts with a **Code node full of mock records**. Swap them when
activating:

| Suite | Replace this mock node | With |
|---|---|---|
| Invoicing | `Mock Invoice Input` | Your practice-management system's "new invoice" webhook |
| Recovery | `Mock Overdue Invoices` | A schedule query against your invoice ledger (Zoho/Tally/QuickBooks/Sheets) |
| GST | `Mock Client GST Calendar` | Your client-return calendar with `dataReceived`, `outputTax`, `inputTaxCredit` |
| Documents | `Mock Pending Document Checklist` | Your document tracker, including `reminderCount` |
| Support | *(nothing to replace)* | Point the WhatsApp/voice webhooks at the given paths |

### 3.4 Activate the webhooks

The support suite exposes three endpoints — activate the workflow, then point
your channels at them:

| Channel | Webhook path | Connect to |
|---|---|---|
| Web chat | *(n8n chat trigger URL)* | Embed n8n's chat widget, or your own UI |
| WhatsApp | `POST /accounting/whatsapp-in` | Meta App dashboard → WhatsApp → Callback URL |
| Voice (Sarvam) | `POST /accounting/voice-in` | Sarvam agent tool → your n8n voice webhook URL |

For WhatsApp, Meta requires webhook verification — set the **Verify token**
in Meta's dashboard and answer the `hub.challenge` handshake (n8n's webhook
node handles GET verification if you enable it, or add a small Respond node).

### 3.5 Mirror the connections in the CRM demo

Open `ledgerpro-crm.html` → **Settings**, and fill the same values so the demo's
messages show your real details. Each card has **Save**, and connections also
have **Test** (simulated) and **Disconnect**:

| Settings card | Fields |
|---|---|
| **Firm Profile** | Firm name, firm email/phone, partner email, team email |
| **Client Links** | Payment link, document upload link |
| **Rules & Thresholds** | Approval limit, default GST %, due days, escalation cuts, reminder limit |
| **Accounting Software** | API key, software (Zoho Books / Tally Prime / QuickBooks) |
| **Email Account** | Sender email, app password / token |
| **WhatsApp Business** | Phone number ID, access token |
| **AI Voice Agent** | API key, org ID, workspace ID, agent app ID, connection ID, agent phone |
| **AI Model** | API key, model (e.g. `gpt-5-mini`) |

There's also an **AI Knowledge Base** section: add comma-separated keywords and
an answer, and the demo assistant uses it before falling back to handoff — the
same pattern the production agent should follow with a real retrieval tool.

### 3.6 Go-live checklist

- [ ] Gmail credential connected on **every** Gmail node (no warning icons)
- [ ] OpenAI credential on the Chat Model node
- [ ] All `YOUR_*` / `yourfirm.example.com` placeholders replaced
- [ ] Mock Code nodes swapped for real data sources
- [ ] WhatsApp webhook verified in Meta's dashboard
- [ ] Sarvam agent tool pointed at the voice webhook
- [ ] Test each suite with one real client record before switching schedules on
- [ ] CRM Settings filled to match — messages and links stay consistent

---

## Security notes

- **Never commit real tokens.** The template ships with placeholders only; the
  one credential *name* from the author's n8n instance (`fitness stiudio`) is
  harmless but should be re-selected on import.
- The CRM keeps everything in `localStorage` on your machine — nothing is sent
  anywhere by the HTML file itself.
- WhatsApp and Sarvam keys live inside HTTP nodes; consider n8n **variables**
  or an environment store if you share the workflow with teammates.
- Gmail "app password" style tokens in the CRM demo are simulated — the demo
  never sends email.

## Tech notes

- CRM: vanilla HTML/CSS/JS + [Chart.js](https://www.chartjs.org/) and
  [Lucide icons](https://lucide.dev) from CDN; no framework, no build step.
- n8n: tested against current node versions in the file (`webhook` 2.1,
  `set` 3.5, `if` 2.3, `switch` 3.4, `gmail` 2.2, LangChain agent 3.1).
- Voice replies are shortened and URL-stripped (`voiceify`) so they read
  naturally when spoken — mirrors what the Sarvam agent should say aloud.

---

Built by **Semontech** — websites, AI voice agents, chatbots, custom CRMs and
dashboards for service businesses.

dev.shahzama@gmail.com · +971 52 611 4643 (WhatsApp)
