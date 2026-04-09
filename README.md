# 🧾 AI-Powered Invoice Processing Automation

> Automatically extract, structure, and archive invoice data from PDFs using n8n, LlamaParse, and Groq — with zero manual data entry.

---

## 📌 Overview

This project automates the end-to-end invoice processing pipeline for professional service firms (e.g., Chartered Accountants, Finance teams). Raw PDF invoices dropped into a Google Drive folder are automatically parsed, structured by an LLM, and logged into a Google Sheet — then archived cleanly.

**Estimated time savings: up to 40% reduction in manual data entry.**

---

## 🏗️ System Architecture

```
[Google Drive: /Unprocessed]
        │
        ▼
[Download File]
        │
        ▼
[LlamaParse API → Upload PDF]
        │
        ▼
[Wait + Poll → Job Status Check]
        │        │
     SUCCESS   PENDING ──► (loop back to Wait)
        │
        ▼
[Fetch Parsed Markdown]
        │
        ▼
[Groq LLM → Information Extractor]
        │
        ▼
[Append Row → Google Sheets]
        │
        ▼
[Move to /Processed → Delete from /Unprocessed]
```

---

## ✨ Features

- 🔍 **High-accuracy PDF parsing** via LlamaParse (handles complex invoice layouts)
- 🤖 **LLM-powered field extraction** using Groq (Kimi K2 model via LangChain)
- 📊 **Auto-logging** to Google Sheets with structured fields
- ♻️ **Robustness loop** — polls until parsing job is complete before proceeding
- 🗂️ **Auto-archiving** — moves processed files and cleans up source folder
- 🐳 **Self-hosted on VPS via Docker** — no per-execution cloud costs

---

## 📦 Tech Stack

| Component | Tool |
|---|---|
| Workflow Automation | [n8n](https://n8n.io/) (self-hosted) |
| PDF Parsing | [LlamaParse](https://cloud.llamaindex.ai/) |
| LLM Inference | [Groq API](https://groq.com/) |
| LLM Model | `moonshotai/kimi-k2-instruct` |
| File Storage | Google Drive |
| Database | Google Sheets |
| Infrastructure | Docker on VPS |

---

## 🔧 Setup & Configuration

### Prerequisites

- n8n instance (self-hosted via Docker recommended)
- Google Cloud project with **Drive API** and **Sheets API** enabled
- [LlamaParse API key](https://cloud.llamaindex.ai/)
- [Groq API key](https://console.groq.com/)

---

### Step 1: Google Cloud Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Enable the **Google Drive API** and **Google Sheets API**
3. Create **OAuth 2.0 credentials** and configure them in n8n

---

### Step 2: Google Drive Structure

Create two folders in your Google Drive:

```
My Drive/
├── Unprocessed/   ← Drop new invoice PDFs here
└── Processed/     ← Archived invoices land here automatically
```

---

### Step 3: Google Sheets Database

Create a Google Sheet with the following column headers:

| invoiceNumber | invoiceDate | vendorName | totalAmount |
|---|---|---|---|

---

### Step 4: Configure Environment Variables

Before importing the workflow, replace all placeholder values in `Invoice_Processing_Workflow_template.json`:

| Placeholder | Description |
|---|---|
| `YOUR_GOOGLE_DRIVE_CREDENTIAL_ID` | n8n credential ID for Google Drive OAuth |
| `YOUR_GOOGLE_SHEETS_CREDENTIAL_ID` | n8n credential ID for Google Sheets OAuth |
| `YOUR_GROQ_CREDENTIAL_ID` | n8n credential ID for Groq API |
| `YOUR_LLAMAPARSE_API_KEY` | Your LlamaParse API key from cloud.llamaindex.ai |
| `YOUR_UNPROCESSED_FOLDER_ID` | Google Drive folder ID for `/Unprocessed` |
| `YOUR_PROCESSED_FOLDER_ID` | Google Drive folder ID for `/Processed` |
| `YOUR_GOOGLE_SHEET_ID` | Google Sheets document ID |
| `YOUR_WORKFLOW_ID` | (Auto-assigned by n8n on import) |
| `YOUR_INSTANCE_ID` | (Auto-assigned by n8n on import) |
| `YOUR_TAG_ID` | (Auto-assigned by n8n on import) |

> ⚠️ **Never commit real API keys or credential IDs to version control.** Use n8n's built-in credential manager to store secrets securely.

---

### Step 5: Import & Activate Workflow

1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Upload `Invoice_Processing_Workflow_template.json`
4. Attach the correct credentials to each node
5. Set the **Google Drive Trigger** folder to your `/Unprocessed` folder
6. Click **Activate**

---

## 📋 Extracted Invoice Fields

The LLM Information Extractor is configured to pull the following fields from each invoice:

| Field | Required | Description |
|---|---|---|
| `invoiceNumber` | ✅ | Invoice ID or number |
| `invoiceDate` | ✅ | Date on the invoice |
| `dueDate` | | Payment due date |
| `vendorName` | ✅ | Supplier/vendor name |
| `vendorAddress` | | Supplier address |
| `clientName` | | Buyer/client name |
| `lineItems` | | Itemized list with qty, unit price, total |
| `subtotal` | | Pre-tax subtotal |
| `taxAmount` | | Tax charged |
| `totalAmount` | ✅ | Final total due |
| `currency` | | Invoice currency |
| `paymentTerms` | | e.g., Net 30 |

> Currently, only `invoiceNumber`, `invoiceDate`, `vendorName`, and `totalAmount` are written to Google Sheets. Extend the Sheets node to log additional fields as needed.

---

## 🔄 Workflow Logic Detail

### Robustness Loop (Status Polling)

LlamaParse jobs are asynchronous. The workflow handles this gracefully:

```
Upload PDF → Wait (5s) → Check Job Status
                              │
                    ┌─────────┴─────────┐
                 SUCCESS             PENDING
                    │                   │
              Fetch Result         Loop back to Wait
```

This ensures the workflow never processes incomplete parse results.

---

## 🚀 Scaling & Future Enhancements

- [ ] **Multi-channel input** — Accept invoices via Telegram/WhatsApp bot
- [ ] **Image support** — Add OCR nodes for JPG/PNG invoices
- [ ] **Error handling** — Slack/email alert on parse failure
- [ ] **Analytics dashboard** — Auto-generate tax liability and margin reports in Google Sheets
- [ ] **Duplicate detection** — Flag invoices with matching numbers before inserting
- [ ] **Multi-currency normalisation** — Convert all totals to a base currency

---

## 💼 Business Value

| Problem | Solution |
|---|---|
| Manual data entry bottleneck | Fully automated extraction pipeline |
| Human error in tax figures | LLM validates and structures all amounts |
| Slow invoice processing | Near real-time processing on file upload |
| No audit trail | Immutable log in Google Sheets + archived originals |

---

## 📁 Repository Structure

```
.
├── Invoice_Processing_Workflow_template.json   # n8n workflow (credentials masked)
└── README.md
```

---

## 🛡️ Security Notes

- All credentials are managed through n8n's encrypted credential store
- No API keys are hardcoded in the workflow JSON — only placeholder strings
- OAuth 2.0 is used for all Google service integrations
- Consider restricting Google Drive API scope to specific folders only

---

## 📄 License

MIT License — free to use, modify, and distribute.

---

## 🙌 Acknowledgements

- [n8n](https://n8n.io/) — Workflow automation platform
- [LlamaIndex / LlamaParse](https://www.llamaindex.ai/) — PDF parsing engine
- [Groq](https://groq.com/) — Ultra-fast LLM inference
- [MoonshotAI Kimi K2](https://moonshotai.github.io/Kimi-K2/) — Language model used for extraction
