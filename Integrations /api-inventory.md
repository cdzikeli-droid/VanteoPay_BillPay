# GOSPDR Bill Pay API Inventory

**Product:** Vanteo Connect — Bill Pay
**Base URL:** `https://api.sb.spidrsrv.com/v1`
**Source:** SPDR (Galileo Bill Pay platform, on behalf of sponsor bank FBOL) — [docs.gospidr.com](https://docs.gospidr.com)
**Purpose:** Master reference of every GOSPDR Bill Pay endpoint referenced across Vanteo Connect PRDs and user stories, with traceability to Jira. Update the Jira Story ID column as stories are created — do not remove an API row without confirming it's no longer used in any active flow.
**Last compiled:** 2026-07-14

> **Traceability gap:** Every PRD reviewed (Part 1, Part 2, and prior drafts) lists Jira Epic/Story as **TBD**. No Bill Pay API currently has a real Jira ID attached. This file uses `TBD` as a placeholder in the Jira column — replace with actual keys as stories are written, or connect your Jira/Atlassian account so Claude can pull real issue IDs automatically.

---

## 1. Paper Check APIs (Part 1 — Mail a Check)

Source of record: `Vanteo_Connect_Bill_Pay_PRD_Part1_PaperCheck.docx` (most detailed spec; supersedes `Vanteo_BillPay_UserStories_v1/v2` and `Vanteo_Connect_BillPay_PRD_v2–v6`, which reference the same endpoints with less parameter detail).

| # | API Name | Method | Endpoint | Key Parameters | Trigger / Screen | Jira Story ID | Notes |
|---|---|---|---|---|---|---|---|
| 1 | Get Billers | GET | `/billpay/getBillers` | `accountId` | Bill Pay home / Landing Page on load | TBD | Shared endpoint — also returns electronic billers (see §2). Returns Active records only; superseded/edited records excluded. |
| 2 | Add Paper Biller | POST | `/billpay/addPaperBiller` | `accountId`, `billerName`, `billerCity`, `billerState`, `billerZip` (required); `billerAddress1`, `billerAddress2`, `billerPhone`, `amount` (optional) | Review & Confirm on Add Payee flow | TBD | Returns `billerId` for immediate use in Create Bill Payment. |
| 3 | Modify Paper Biller | POST | `/billpay/modifyPaperBiller` | `accountId`, `billerId` (existing), `billerName`, `billerCity`, `billerState`, `billerZip` (required); address2/phone (optional) | Save Changes on Edit Payee | TBD | Creates a **new** payee record + new `billerId`; original record preserved (not deleted) for payment history integrity. |
| 4 | Remove Biller | POST | `/billpay/removeBiller` | `accountId`, `billerId` | Confirm on Remove Payee dialog | TBD | Does **not** cancel pending payments already scheduled against that biller. |
| 5 | Create Bill Payment | POST | `/billpay/createPayment` | `accountId`, `billerId`, `amount`, `processDate` (YYYY-MM-DD), `memo` (optional, max 50 chars) | Submit on Review & Confirm Payment | TBD | Shared endpoint — also used for electronic/ACH payments (see §2); payment method inferred from biller type. |
| 6 | Cancel Payment | POST | `/billpay/cancelPayment` | `accountId`, `billpayRequestId` | Confirm Cancel on payment detail screen | TBD | Only works on payments not yet processed. |
| 7 | Get Payment History | GET | `/billpay/getPaymentHistory` | `accountId`, `startDate`, `endDate` (YYYY-MM-DD) | Payment History screen on load / date range change | TBD | Default range: last 90 days. |
| 8 | Get Scheduled Payments | GET | `/billpay/getScheduledPayments` | Not fully specified in source docs | Bill Pay home on load (paired with Get Billers) | TBD | **Referenced in user flow only** — no dedicated parameter/notes section exists in any PRD. Needs a full spec before implementation. |

---

## 2. Electronic / ACH Biller APIs (Part 2 — Pay a Biller)

Source of record: `Vanteo_Connect_Bill_Pay_PRD_Part2_ElectronicBiller.docx` (most current; supersedes `Vanteo_Connect_BillPay_ElectronicBillers_PRD_v1` and `Vanteo_BillPay_ElectronicBillers_UserStories_v1–v3`).

| # | API Name | Method | Endpoint | Key Parameters | Trigger / Screen | Jira Story ID | Notes |
|---|---|---|---|---|---|---|---|
| 9 | Get Billers (electronic filter) | GET | `/billpay/getBillers` | `accountId`, `billerType=electronic` | List of Billers on load | TBD | Same endpoint as #1, filtered by type. |
| 10 | Search Billers (RPPS directory) | GET | `/billpay/searchBillers` | `query` (biller name) | Select a Biller, as user types | TBD | Searches Vanteo's registered biller directory (RPPS network). |
| 11 | Add Electronic Biller | POST | `/billpay/addElectronicBiller` | `accountId`, `billerId` (from registry), `customerAccountNumber` (required), `nickname` (optional) | Save & Link on Create Biller Customer Link | TBD | Returns a Vanteo `billerId` for use in Create Bill Payment. |
| — | Create Bill Payment | POST | `/billpay/createPayment` | See #5 | Submit on Review & Confirm Payment | TBD | Same endpoint as #5 — no separate API. |

---

## 3. Superseded / Draft Naming (needs reconciliation)

These endpoint names appear in earlier drafts but were replaced by the names above in the current Part 1 / Part 2 PRDs. Flagging so engineering can confirm which name is correct before building — a common source of broken traceability if both names end up referenced in different Jira stories.

| Draft Name | Found In | Likely Current Equivalent | Status |
|---|---|---|---|
| `POST /billpay/searchBillerDirectory` | `Vanteo_Connect_BillPay_ElectronicBillers_PRD_v1`, `Vanteo_BillPay_ElectronicBillers_UserStories_v1–v3` | `GET /billpay/searchBillers` (#10) | Needs confirmation — unclear if renamed or a distinct endpoint |
| `POST /billpay/linkBiller` *(marked "TBD" in source, endpoint name unconfirmed)* | `Vanteo_Connect_BillPay_ElectronicBillers_PRD_v1` | `POST /billpay/addElectronicBiller` (#11) | Needs confirmation |

---

## 4. Open Engineering Questions Affecting This Inventory

Pulled from PRD "Open Questions" sections — resolving these may change parameters or add fields above.

| API | Question | Owner |
|---|---|---|
| Add Paper Biller | Does it return a `billerId` synchronously, or is there an async step before Create Bill Payment can be called? | Engineering |
| Modify Paper Biller | Does it return the new `billerId` synchronously? Does it set the original record to Inactive? | Engineering |
| All write operations | Is ZTM (Zero Trust / session token) enabled? If so, `sessionKey` is required on all POST endpoints. | Engineering |
| Get Billers | Does it return Part 1 paper payees and Part 2 electronic billers in one combined response, or via separate calls? | Engineering |
| Add Electronic Biller / Search Billers | Is a ZTM session key required here too, consistent with the paper flow's open question? | Engineering |

---

## How to Use This File

1. As each API gets a real Jira story, replace `TBD` in the Jira Story ID column with the issue key (e.g. `BP-142`).
2. When an endpoint name is confirmed or changed by engineering, update the row and move the old name into §3 with a "Deprecated" status rather than deleting it — preserves traceability history.
3. Before adding a new API to this file, check whether it already exists under a different draft name (see §3) to avoid duplicate tracking.
test
