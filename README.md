# 📧 Power Automate – Shared Inbox Invoice Classifier

![Power Automate](https://img.shields.io/badge/Power%20Automate-0066FF?style=for-the-badge&logo=microsoft&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active%20in%20Production-brightgreen?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Finance%20Automation-blueviolet?style=for-the-badge)

## 📌 Overview

This Power Automate flow monitors a **shared mailbox** and automatically classifies incoming emails as invoices or non-invoices. If an invoice is detected, it identifies the invoice type and routes the email to the corresponding folder — eliminating manual sorting and reducing processing time.

> **Business Impact:** Reduces manual email triage in Accounts Receivable operations, improving speed and accuracy of invoice intake.

---

## ⚙️ How It Works

### Flow Architecture

```
Trigger (New Email)
    │
    ▼
HTML to Text Conversion
    │
    ▼
Compose Body Text
    │
    ▼
Compose Split Words  (keyword tokenization)
    │
    ▼
Initialize Variables  (invoice type flags)
    │
    ▼
Evaluate if Invoice Type A  ──► Move to Type A Folder
    │
    ▼
Evaluate if Invoice Type B  ──► Move to Type B Folder
    │
    ▼
Condition (fallback check)  ──► Default handling
```

### Step-by-Step Logic

| Step | Action | Description |
|------|--------|-------------|
| 1 | **Trigger** | Fires when a new email arrives in the shared mailbox |
| 2 | **HTML to Text** | Strips HTML formatting from the email body for clean text processing |
| 3 | **Compose BodyText** | Extracts the plain text body for analysis |
| 4 | **Compose SplitWords** | Tokenizes the body into keywords for pattern matching |
| 5 | **Initialize Variables** | Sets up boolean/string variables to hold classification results |
| 6 | **Evaluate if Invoice Type A** | Checks for keywords associated with Invoice Type A (2 cases: match / no match) |
| 7 | **Evaluate if Invoice Type B** | Checks for keywords associated with Invoice Type B (2 cases: match / no match) |
| 8 | **Condition (Fallback)** | Final catch-all condition for unclassified or non-invoice emails |

---

## 📂 Folder Structure

```
power-automate-invoice-classifier/
│
├── README.md
├── docs/
│   └── flow-overview.png        ← Visual diagram of the full flow
└── flow-logic/
    └── flow-description.md      ← Detailed technical description
```

---

## 🔧 Technologies Used

- **Microsoft Power Automate** – Flow automation
- **Microsoft Outlook / Shared Mailbox** – Email trigger and folder management
- **Power Automate Expressions** – HTML parsing, string manipulation, conditional logic

---

## 💡 Key Expressions Used

```plaintext
# Convert HTML body to plain text
body('Html_to_text')?['text']

# Split body into keywords
split(outputs('Compose_BodyText'), ' ')

# Check if keyword exists in body
contains(outputs('Compose_SplitWords'), 'keyword')
```

---

## 🎯 Use Case

This flow was built for **Accounts Receivable** teams that receive high volumes of vendor or customer emails in a shared inbox. Instead of manually reading and sorting each email, the flow:

- ✅ Detects invoices automatically based on keywords
- ✅ Classifies the invoice type (e.g., by vendor, system, or document type)
- ✅ Routes the email to the correct subfolder in real time
- ✅ Reduces human error and processing time

---

## 👤 Author

**David** – Cash Application Analyst | Power Automate Developer  
📍 Costa Rica  
🔗 [GitHub Profile](https://github.com/YOUR_USERNAME)

---

## 📄 License
