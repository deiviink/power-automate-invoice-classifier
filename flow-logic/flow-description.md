# Flow Technical Description
## Power Automate – Shared Inbox Invoice Classifier

---

## 1. Trigger

**Action:** `When a new email arrives in a shared mailbox (V2)`

| Parameter | Value |
|-----------|-------|
| Mailbox address | Shared inbox (finance operations) |
| Folder | Inbox |
| Include attachments | No |
| Only with attachments | No |

The flow fires on every new incoming email. No filters at trigger level — all classification logic is handled downstream.

---

## 2. HTML to Text

**Action:** `Html to text`

```
Input: triggerOutputs()?['body/body']
```

Strips all HTML tags from the email body and returns clean plain text. This is necessary because most email clients send rich HTML content — without this step, expressions like `contains()` would match against raw HTML markup instead of readable text.

---

## 3. Compose BodyText

**Action:** `Compose`

```
Input: body('Html_to_text')?['text']
```

Stores the clean plain text as a named output (`BodyText`) for reuse in subsequent steps. Avoids calling the HTML-to-text output multiple times.

---

## 4. Compose SplitWords

**Action:** `Compose`

```
Input: split(outputs('Compose_BodyText'), ' ')
```

Converts the plain text body into an array of individual words by splitting on spaces. This enables precise keyword matching in later steps.

**Why split instead of searching the full string?**

Using `contains()` on the raw string could produce false positives — for example, searching for `"SAP"` could match words like `"SAPPHIRE"`. Splitting into tokens and matching exact words increases accuracy.

---

## 5. Initialize Variables

Two variables are initialized to track the classification result:

| Variable | Type | Initial value | Purpose |
|----------|------|---------------|---------|
| `varInvoiceType` | String | `""` | Stores detected invoice type |
| `varIsInvoice` | Boolean | `false` | Flags whether email is an invoice |

---

## 6. Evaluate if Invoice Type A

**Action:** `Switch` (2 cases)

```
Expression: contains(outputs('Compose_SplitWords'), 'KEYWORD_A')
```

| Case | Condition | Action |
|------|-----------|--------|
| True | Keyword A found in body | Set `varInvoiceType = "Type A"`, move email to Type A folder |
| False | Keyword A not found | Continue to next evaluation |

---

## 7. Evaluate if Invoice Type B

**Action:** `Switch` (2 cases)

```
Expression: contains(outputs('Compose_SplitWords'), 'KEYWORD_B')
```

| Case | Condition | Action |
|------|-----------|--------|
| True | Keyword B found in body | Set `varInvoiceType = "Type B"`, move email to Type B folder |
| False | Keyword B not found | Continue to fallback condition |

---

## 8. Condition (Fallback)

**Action:** `Condition`

Handles emails that did not match any invoice type. Two branches:

- **Yes branch:** Email matched a secondary rule or requires manual review — moves to a default folder
- **No branch:** Not an invoice — no action or alternate routing

---

## Key Expressions Reference

```plaintext
# Clean email body
body('Html_to_text')?['text']

# Tokenize body
split(outputs('Compose_BodyText'), ' ')

# Check for keyword
contains(outputs('Compose_SplitWords'), 'KEYWORD')

# Move email to folder
Office 365 Outlook: Move email (V2)
  - Message Id: triggerOutputs()?['body/id']
  - Folder: /Inbox/Invoices/Type A
```

---

## Limitations & Known Considerations

- Keywords are **case-sensitive** by default in Power Automate expressions. Consider using `toLower()` for more robust matching:
  ```
  contains(split(toLower(outputs('Compose_BodyText')), ' '), 'keyword')
  ```
- The `split(' ')` approach does not handle punctuation — a word followed by a comma (`"invoice,"`) will not match `"invoice"`. A future improvement could use regex-based parsing.
- Currently evaluates one keyword per invoice type. Multiple keywords per type would require an `OR` condition or an array-based approach.

---

## Potential Improvements

- [ ] Add `toLower()` for case-insensitive matching
- [ ] Support multiple keywords per invoice type using `intersection()`
- [ ] Log unclassified emails to a SharePoint list for manual review
- [ ] Add email reply confirmation when an invoice is successfully classified
- [ ] Extend classification to read subject line in addition to body

---

## Environment

| Component | Details |
|-----------|---------|
| Platform | Microsoft Power Automate |
| Connector | Office 365 Outlook |
| Trigger type | Automated cloud flow |
| Status | Active in production |
