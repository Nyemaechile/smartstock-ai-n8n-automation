# SmartStock AI — Inventory & Restocking Automation (n8n)

A hands-on **three-workflow inventory automation prototype**, designed and configured using **n8n Cloud, Google Sheets, JavaScript, and Gmail**. Built as a learning and portfolio case study using sample inventory records; it is **not a production-ready procurement system**.

## Business problem
Manual stock records make it easy to lose track of item movements, approve an invalid stock-out, or overlook products that need replenishment. This prototype connects stock entry, restocking recommendations, notifications, and a basic purchase-decision form.

## The three workflows

| Workflow | Purpose | Main steps |
|---|---|---|
| [01 — Inventory Entry](workflows/01-inventory-entry.json) | Capture stock-in/out, validate and update balances | n8n Form → Google Sheets lookup → JavaScript validation → valid/rejected branch → transaction log and low-stock alert |
| [02 — Restocking Intelligence](workflows/02-restocking-intelligence.json) | Recommend how much stock to reorder | Manual/daily trigger → read inventory → JavaScript calculation → filter → upsert recommendations → check prior email → Gmail notification → delivery flag |
| [03 — Purchase Approval](workflows/03-purchase-approval.json) | Record a simulated manager approval/rejection | n8n Form → pending recommendation lookup → JavaScript validation → IF gate → Google Sheets update |

## Inventory logic
- Reject non-positive or non-integer transaction quantities.
- Reject stock-out requests that exceed current stock.
- Classify stock as LOW STOCK when `currentStock <= minimumStock`.
- Set restocking target to `2 × minimumStock` (a **demo rule**, not a forecasting model).
- Suggested reorder quantity: `max(0, ceil(targetStock - currentStock))`.
- Classify restocking priority: CRITICAL (zero), HIGH (at most half minimum), MEDIUM (at most minimum), NORMAL (above minimum).
- Use a `Product ID|Branch|PENDING` key to update an existing pending recommendation and a `Notification Sent` field to reduce repeat emails.

## Example walkthrough (synthetic data)
1. Stapler in Lagos has **8** units against a minimum of **15**.
2. Analysis calculates target **30** and recommended order **22**.
3. Recommendation is stored as **PENDING** and an email can notify the test manager.
4. A purchase-approval test records a decision of **APPROVED**, quantity **22**, manager, timestamp, and remarks.
5. **Approval does not add stock.** Goods received must be recorded separately through Stock In.

## Repository layout
```
workflows/
  01-inventory-entry.json
  02-restocking-intelligence.json
  03-purchase-approval.json
sample-data/
  stock_balance.csv
  transactions.csv
  restocking_recommendations.csv
  rejected_transactions.csv
README.md
.gitignore
```

## Run it yourself (n8n)
1. Create your own Google spreadsheet with the four sheets represented by the sample CSVs. Import each CSV into a **separate worksheet** named `Stock_Balance`, `Transactions`, `Restocking_Recommendations`, and `Rejected_Transactions` respectively. Pay attention to column names, including spaces: the exported mappings reflect the prototype's sheet headers. Re-check each node's column mapping after setup.
2. In n8n, create or open a workflow, choose **Import from File**, and import one JSON from `workflows/`.
3. Reconnect the **Google Sheets OAuth** credential and choose **your own spreadsheet** and correct worksheet in each Google Sheets node. The shared copies deliberately contain a placeholder spreadsheet ID and no credentials.
4. Reconnect Gmail in the notification nodes and replace `YOUR_EMAIL@example.com` with a test inbox you control.
5. Use the **test form URL** and test with synthetic entries. Keep all workflows inactive until you validate the branches and spreadsheet updates. Do not use a real manager name to authenticate anyone.
6. Set the workflow timezone for the Schedule Trigger, and publish only when ready. Note that cloud trial availability depends on the hosting plan.

> **Import note:** Some sheet selectors are represented using worksheet names rather than internal IDs for privacy. Reselect each spreadsheet and sheet in n8n if a node reports an invalid resource selector. These exports preserve workflow logic, but are not plug-and-play until credentials and sheets are reconfigured.

## Scope and limitations
- **Prototype, not actual machine learning/AI**: “AI” is the project's name. The supplied workflow logic uses deterministic rules and JavaScript; there is no trained AI model.
- Manager Name on the approval form is **self-reported**. Authentication/authorization must be added before real management use.
- A completed approval is not the same as an actual purchase or goods receipt.
- Manually clearing the pending key after approval was part of testing; the exported approval workflow still uses that key to match the row. Production use needs reliable closure and checks against approved-but-undelivered orders.
- Repeated recommendations, parallel stock transactions, incomplete lookup results, and permission/security edge cases need further hardening before deployment.
- Test with non-sensitive records. Sample CSVs contain **fictional data**.

## Privacy and publishing
Workflow files have been prepared for public GitHub sharing: OAuth credential objects, workflow/webhook identifiers at the export level, live spreadsheet IDs/URLs and personal notification addresses have been removed or replaced. Review again before publication, especially if adding new screenshots, URLs, forms, or credentials.

## Built with
- n8n Cloud
- Google Sheets
- Gmail
- JavaScript (n8n Code node)

## Credit
Designed, assembled, configured, debugged, and tested as a personal n8n learning project, with AI-assisted guidance during development.
