# PRD Overview

By Christina Zikeli

## Document Control

**Feature Name:** Bill Pay — Part 3: Recurring Payments, Payee/Biller Management, ACH Cancellation & Bill Presentment
**Product Area:** Move Money
**Target Release:** Phase 2 / Phase 3 (Next Phase — builds on Part 1 and Part 2)
**PRD ID:** P3 *(part of the Bill Pay PRD family: P1 = Part 1 Mail a Check, P2 = Part 2 Pay a Biller/ACH)*

### Change Log

| Version | Date | Author | Summary |
|---|---|---|---|
| 1.0 | July 14, 2026 | Christina Zikeli | Initial draft — pulls in the four capabilities explicitly deferred in Part 1 and Part 2: recurring/scheduled payments, delete a saved payee / unlink a biller, cancel a submitted electronic payment, and bill presentment. |

---

## 1. Overview

Part 1 (Mail a Check) and Part 2 (Pay a Biller / ACH) both deliberately deferred several capabilities to keep initial scope small. This document specs out those deferred items now that Part 1 and Part 2 are in build:

| Capability | Deferred from | Original scope note |
|---|---|---|
| Recurring / scheduled payments | Part 1 §1.2, Part 2 Out of Scope | "Recurring payment (weekly / monthly / quarterly / yearly) — No, Part 2" (Part 1); "Recurring or scheduled electronic payments — Deferred to a later phase" (Part 2) |
| Delete a saved payee / Remove-unlink a biller | Part 1 §1.2 ("Delete a saved payee — Phase 3"), Part 2 Out of Scope ("Remove / unlink a biller," "Edit a biller link") | Both parts explicitly punted removal/unlinking to a later phase |
| Cancel a submitted electronic payment | Part 2 Out of Scope | "Cancel a submitted electronic payment — Deferred to a later phase" |
| Bill presentment | Part 2 Out of Scope | "Bill presentment (viewing the biller's actual invoice/amount due) — Out of scope; the user manually enters the payment amount" |

This document also formally specs **Get Scheduled Payments**, which was referenced in Part 1's user flow but never given a full parameter/response spec (see Part 1 Action Item #6 in `BillPay_Action_Items.md`), and gives **Remove Biller** (paper check) its first Jira-visible home (Action Item #1).

### In Scope

| Area | Description |
|---|---|
| Schedule a Payment | Add frequency (weekly / monthly / quarterly / yearly), start date, and optional end date to paper check and ACH payments. |
| Manage Scheduled Payments | View, edit, and cancel an upcoming (not-yet-processed) instance of a recurring or future-dated payment. |
| Delete a Saved Payee (paper check) | Formalize and give Jira coverage to the existing `removeBiller` API for paper check payees. |
| Remove / Unlink an Electronic Biller | Equivalent removal capability for ACH-linked billers. |
| Edit a Biller Link | Update the customer account number on an existing electronic biller link. |
| Cancel a Submitted Electronic (ACH) Payment | Extend cancellation to ACH payments, within whatever window the faster settlement time allows. |
| Bill Presentment | Display the biller's current amount due and due date when the biller (via RPPS) supports it, so the user doesn't have to manually enter the amount. |

### Out of Scope

| Area | Description |
|---|---|
| Recurring payments to a new (not-yet-added) payee/biller in the same flow | User must add the payee/biller first, then schedule recurring payments — no combined "add + schedule" flow in this phase. |
| Partial bill presentment (partial payment of a presented invoice) | User can only pay the full presented amount or manually enter a different amount; splitting a bill is out of scope. |
| Multi-account selection for recurring payments | Consistent with Part 1/Part 2 — a single DDA account per scheduled series. |

---

## 2. Use Cases

**Recurring payment:** Priya pays her roommate $500 every month via paper check (set up in Part 1). Instead of re-entering the payment manually each month, she sets up a monthly recurring payment from Payee Details, so Vanteo automatically mails a check on the same day each month until she cancels.

**Delete a payee / unlink a biller:** Priya's roommate situation ends. She deletes the paper check payee so it no longer appears in her list. Separately, Deepak (from Part 2) switches electric providers and unlinks his old electric biller.

**Cancel a submitted ACH payment:** Deepak submits an ACH payment to his electric company but realizes he entered the wrong amount five minutes later. Because ACH settles in 1–3 business days, he may still have a window to cancel before it posts.

**Bill presentment:** Deepak's electric company (via RPPS) publishes its current bill amount and due date. Instead of manually typing in the amount, Deepak sees "Amount Due: $87.42, Due June 30" pre-filled on the Pay a Biller screen and can accept it or override it.

### Key Takeaways

| Insight | PRD Implication |
|---|---|
| Recurring payments apply to both payment types (paper + ACH) | One shared "Schedule a Payment" experience should extend both Create a Payment (Part 1) and Pay a Biller (Part 2), rather than building two separate flows. |
| Deleting/unlinking doesn't cancel payments already in flight | Consistent with Part 1's existing Remove Biller behavior — recurring cancellation and payee/biller deletion are separate actions with separate confirmations. |
| ACH cancellation has a narrow, uncertain window | Because ACH settles in 1–3 business days (Part 2 assumption), the cancellation window may be very short or may not exist once processing starts — this needs an explicit Open Question to Engineering/Ops (see §11). |
| Bill presentment depends on the biller, not on Vanteo | Not every RPPS-registered biller will support presentment — the UI must gracefully fall back to manual amount entry when it's unavailable. |

---

## 3. Assumptions

- Primary users remain foreign nationals on temporary work or student visas, consistent with Part 1 and Part 2.
- Recurring payment frequency options are weekly, monthly, quarterly, and yearly — matching the `frequencyType` values referenced in earlier Bill Pay drafts (see `Vanteo_Connect_BillPay_ElectronicBillers_PRD_v1`).
- A recurring series can be canceled by the user at any time; canceling stops future occurrences but does not reverse payments already processed.
- Bill presentment data (amount due, due date) is sourced from the RPPS biller directory via SPDR — Vanteo does not build its own biller invoice storage.
- ACH cancellation is only possible before a payment enters processing; the exact cutoff is TBD with Engineering/Ops (see Open Questions).
- Deleting a payee or unlinking a biller does not delete payment history — consistent with the "old record preserved" pattern from Part 1's Modify Paper Biller.

---

## 4. UI Screen Inventory

| # | Screen Name | Purpose | Entry Point |
|---|---|---|---|
| 1 | Schedule a Payment (frequency step) | Extends Create a Payment (Part 1) / Pay a Biller (Part 2) with frequency, start date, and optional end date fields | Create a Payment or Pay a Biller — "Payment Frequency" control |
| 2 | Manage Scheduled Payments | View all upcoming (not-yet-processed) payments, one-time or recurring, across both payment types | Bill Pay Landing Page — new "Upcoming" tab (currently greyed out per Part 1 FR-08) |
| 3 | Delete a Payee (confirmation) | Confirm and remove a saved paper check payee | Payee Details — Delete button (currently visible but inactive per Part 1 FR-20) |
| 4 | Unlink a Biller (confirmation) | Confirm and remove a linked electronic biller | Biller Details / List of Billers row |
| 5 | Edit a Biller Link | Update the customer account number on an existing electronic biller link | Biller Details — Edit button |
| 6 | Cancel a Submitted Payment (ACH) | Confirm cancellation of a not-yet-processed ACH payment | Payment/History detail — Cancel button |
| 7 | Biller Invoice Preview (Bill Presentment) | Show the biller's current amount due and due date, when available | Pay a Biller — above the Amount field |

---

## 5. User Flow

### Schedule a Recurring Payment

| Step | User Action | System Response |
|---|---|---|
| 1 | On Create a Payment / Pay a Biller, select a frequency other than One-Time. | Reveal Start Date and optional End Date fields. |
| 2 | Enter amount, frequency, start date, optional end date; tap Review. | Validate fields; navigate to Review & Confirm Payment showing the recurring schedule summary. |
| 3 | Confirm on Review & Confirm Payment. | Call `POST /billpay/createPayment` with `frequencyType`, `startDate`, `endDate` (optional). |
| 4 | View success screen. | Confirmation includes next payment date and frequency. |

### Manage Scheduled Payments

| Step | User Action | System Response |
|---|---|---|
| 1 | Tap Upcoming tab on Bill Pay Landing Page. | Call `GET /billpay/getScheduledPayments`; display list of upcoming one-time and recurring payments. |
| 2 | Tap a scheduled payment. | Show detail: payee/biller, amount, frequency, next date, end date (if set). |
| 3 | Tap Cancel Series (recurring) or Cancel Payment (one-time, future-dated). | Confirm dialog; on confirm, call `POST /billpay/cancelPayment`. |
| 4 | Cancellation confirmed. | Remove from Upcoming list; does not affect already-processed payments. |

### Delete a Payee / Unlink a Biller

| Step | User Action | System Response |
|---|---|---|
| 1 | From Payee Details (paper) or Biller Details (ACH), tap Delete / Unlink. | Show confirmation: "Removing this [payee/biller] will not cancel any pending payments. Continue?" |
| 2 | Confirm. | Call `POST /billpay/removeBiller` (paper) or the new electronic-biller removal endpoint (ACH — see Open Questions). |
| 3 | Removal confirmed. | Payee/biller removed from list; payment history retained. |

### Cancel a Submitted ACH Payment

| Step | User Action | System Response |
|---|---|---|
| 1 | From payment detail (History or Upcoming), tap Cancel. | Show confirmation, including a warning if the cancellation window may have already closed. |
| 2 | Confirm. | Call `POST /billpay/cancelPayment` with `billpayRequestId`. |
| 3 | Result. | On success, show canceled state. On failure (already processed), show: "This payment could not be canceled — it may have already posted." |

### Bill Presentment

| Step | User Action | System Response |
|---|---|---|
| 1 | User opens Pay a Biller for a linked biller. | App calls the biller invoice lookup (see Open Questions — endpoint TBC). |
| 2 | If presentment data is available. | Show "Amount Due: $X, Due [date]" pre-filled in the Amount field, editable. |
| 3 | If presentment data is not available. | Fall back silently to the existing manual Amount entry (Part 2 behavior) — no error shown to the user. |

---

## 6. Functional Requirements

*(Using a document-prefixed ID scheme — `P3-FR-##` — to avoid collision with Part 1's and Part 2's independently-numbered FR lists; see Action Item #9 in `BillPay_Action_Items.md`.)*

### Schedule a Payment

| ID | Requirement | Priority |
|---|---|---|
| P3-FR-01 | Payment Frequency control shall offer One-Time, Weekly, Monthly, Quarterly, and Yearly. | High |
| P3-FR-02 | Selecting a recurring frequency shall reveal a required Start Date field and an optional End Date field. | High |
| P3-FR-03 | Start Date shall default to today and shall not allow a past date. | High |
| P3-FR-04 | If no End Date is set, the series shall continue until the user cancels it. | Medium |
| P3-FR-05 | Review & Confirm Payment shall display the full recurring schedule (frequency, start date, end date if set) before submission. | High |

### Manage Scheduled Payments

| ID | Requirement | Priority |
|---|---|---|
| P3-FR-06 | App shall call `GET /billpay/getScheduledPayments` to populate the Upcoming tab, replacing its current greyed-out state from Part 1 FR-08. | High |
| P3-FR-07 | Each row shall display payee/biller name, amount, frequency (if recurring), and next payment date. | High |
| P3-FR-08 | Tapping a row shall show full detail and offer a Cancel action. | High |
| P3-FR-09 | Canceling a recurring series shall stop all future occurrences without affecting already-processed payments. | High |

### Delete a Payee / Unlink a Biller

| ID | Requirement | Priority |
|---|---|---|
| P3-FR-10 | Delete (paper check) shall call `POST /billpay/removeBiller`, reusing the existing Part 1 API. | High |
| P3-FR-11 | Unlink (electronic biller) shall call the equivalent removal endpoint for linked billers (endpoint TBC — see Open Questions). | High |
| P3-FR-12 | Both actions shall require a confirmation dialog warning that pending payments are not canceled. | High |
| P3-FR-13 | Deleted/unlinked records shall remain queryable for historical payment display, consistent with Part 1's versioning pattern. | Medium |

### Edit a Biller Link

| ID | Requirement | Priority |
|---|---|---|
| P3-FR-14 | App shall allow editing the customer account number on an existing electronic biller link. | Medium |
| P3-FR-15 | Editing the account number shall require re-confirmation before saving, consistent with Part 1's Edit a Payee pattern. | Medium |

### Cancel a Submitted Electronic Payment

| ID | Requirement | Priority |
|---|---|---|
| P3-FR-16 | App shall offer a Cancel action on any ACH payment not yet processed. | High |
| P3-FR-17 | App shall call `POST /billpay/cancelPayment` with the payment's `billpayRequestId`. | High |
| P3-FR-18 | On failure (payment already processed), app shall show an inline message indicating the payment may have already posted. | High |

### Bill Presentment

| ID | Requirement | Priority |
|---|---|---|
| P3-FR-19 | When presentment data is available for a linked biller, app shall display amount due and due date above the Amount field on Pay a Biller. | Medium |
| P3-FR-20 | Presented amount shall be editable — the user can override it with a different amount. | High |
| P3-FR-21 | When presentment data is unavailable, app shall fall back to manual amount entry with no visible error. | High |

---

## 7. Data Model / Versioning

| Field / Object | Current State (Part 1 / Part 2) | New State (Part 3) | Notes |
|---|---|---|---|
| Payment Frequency | `frequencyType = one_time` only | New values: `weekly`, `monthly`, `quarterly`, `yearly` | Matches `frequencyType` values referenced in earlier ACH drafts. |
| Scheduled Payment Record | Not exposed to the user (referenced only in flow, never fully specified) | New first-class object returned by `getScheduledPayments`: series ID, payee/biller ID, amount, frequency, next date, end date, status | Closes the long-standing spec gap flagged in Action Item #6. |
| Payee / Biller Record Status | `Active` / superseded-on-edit (Part 1), `Active` only (Part 2) | Add `Removed` status | Removed records excluded from active lists but retained for payment history joins. |
| Payment Record (ACH) | No cancellation path (Out of Scope in Part 2) | Add `Canceled` status | Mirrors the paper check payment status model where applicable. |
| Biller Invoice Data | Does not exist | New, read-only, sourced from RPPS via SPDR: amount due, due date | Not stored by Vanteo — fetched live each time Pay a Biller loads. |

---

## 8. API Reference

| Method | Endpoint | Trigger | Key Parameters | Notes |
|---|---|---|---|---|
| GET | `/billpay/getScheduledPayments` | Upcoming tab on load | `accountId` | **Newly fully specified here** — previously referenced only in Part 1's user flow with no parameter/response detail (Action Item #6). Response should include series ID, payee/biller ID, amount, frequency, next date, end date, status. |
| POST | `/billpay/createPayment` | Submit on Review & Confirm Payment (recurring) | `accountId`, `billerId`, `amount`, `frequencyType` (weekly/monthly/quarterly/yearly), `startDate`, `endDate` (optional), `memo` (optional) | Extends the existing Part 1/Part 2 endpoint — same call, additional recurring parameters. |
| POST | `/billpay/removeBiller` | Confirm Delete on Payee Details (paper) | `accountId`, `billerId` | Reuses Part 1's existing endpoint — this PRD is what gives it Jira coverage (Action Item #1). |
| POST | `/billpay/removeElectronicBiller` *(name TBC)* | Confirm Unlink on Biller Details (ACH) | `accountId`, `billerId` | **Endpoint does not yet appear in any SPDR documentation reviewed.** Confirm with Engineering / SPDR docs whether this exists, or whether `removeBiller` is generic across both biller types. |
| POST | `/billpay/modifyElectronicBiller` *(name TBC)* | Save on Edit a Biller Link | `accountId`, `billerId`, `customerAccountNumber` | **Not yet documented anywhere.** Confirm with Engineering whether SPDR supports modifying a biller link, or whether this requires unlink + re-add. |
| POST | `/billpay/cancelPayment` | Confirm Cancel on Cancel a Submitted Payment (ACH) | `accountId`, `billpayRequestId` | Reuses Part 1's existing endpoint. Confirm the cancellation window is meaningful given ACH's 1–3 business day settlement (see Open Questions). |
| GET | `/billpay/getBillerInvoice` *(name TBC)* | Pay a Biller on load, for linked billers | `accountId`, `billerId` | **Speculative — no confirmed SPDR endpoint for bill presentment was found in any source document.** This is the highest-risk item in this PRD; needs direct confirmation from SPDR/Engineering on whether presentment data is exposed at all before this ships. |

Base URL: `https://api.sb.spidrsrv.com/v1`

---

## 9. Error States

| Scenario | User Message | Action |
|---|---|---|
| `getScheduledPayments` fails | Unable to load upcoming payments. Pull to refresh. | Pull-to-refresh on Upcoming tab |
| Recurring payment creation fails | Payment could not be scheduled. Please try again. | Retry on Review & Confirm Payment |
| Delete/unlink fails | We couldn't remove this [payee/biller]. Please try again. | Retry on confirmation dialog |
| Cancel (ACH) fails — already processed | This payment may have already posted and can't be canceled. | No retry; suggest contacting support |
| Biller invoice lookup fails or returns no data | *(No visible error)* | Silently fall back to manual amount entry |
| Edit biller link fails | We couldn't update this account number. Please try again. | Retry on Edit a Biller Link |

---

## 10. Mobile App Navigation

### Navigation Rules

- The Upcoming tab, greyed out since Part 1 (FR-08), becomes active in this phase.
- Delete (paper check) and Unlink (electronic biller) always require a confirmation dialog before the call is made.
- Canceling a recurring series returns the user to Manage Scheduled Payments; canceling a one-time ACH payment returns to Payment History.

### Navigation Map

| Trigger | From Screen | To Screen |
|---|---|---|
| Upcoming tab | Bill Pay Landing Page | Manage Scheduled Payments |
| Scheduled payment row tap | Manage Scheduled Payments | Scheduled Payment Detail |
| Cancel Series / Cancel Payment | Scheduled Payment Detail | Manage Scheduled Payments (row removed) |
| Delete button | Payee Details | Delete a Payee (confirmation) |
| Unlink button | Biller Details | Unlink a Biller (confirmation) |
| Edit Account Number | Biller Details | Edit a Biller Link |
| Cancel button | Payment Detail (ACH, unprocessed) | Cancel a Submitted Payment (confirmation) |

---

## 11. Open Questions

| # | Question | Owner | Status |
|---|---|---|---|
| 1 | Does SPDR expose a dedicated endpoint for unlinking an electronic biller, or is `removeBiller` generic across paper and electronic biller types? | Engineering | Open |
| 2 | Does SPDR support modifying an electronic biller link's account number directly, or must the user unlink and re-add? | Engineering | Open |
| 3 | What is the actual cancellation cutoff for an ACH payment, given 1–3 business day settlement? Is there a meaningful window at all? | Engineering / Banking Partner | Open |
| 4 | **Does GOSPDR/SPDR support bill presentment (biller-provided amount due and due date) at all?** No source document reviewed confirms this endpoint exists. This should be confirmed before any further design or estimation work on this capability. | Engineering / SPDR Account Team | Open — highest risk item in this PRD |
| 5 | Should recurring payment end dates be required, or is "continue until canceled" acceptable as a permanent default? | Product | Open |
| 6 | Does `getScheduledPayments` return recurring series and one-time future-dated payments in a single response, or via separate calls? | Engineering | Open |
| 7 | Should deleting a payee/unlinking a biller also prompt the user to cancel any pending scheduled payments tied to it, or leave them in flight (consistent with Part 1's existing Remove Biller behavior)? | Product / Ops | Open |

---

## 12. Jira Links

| Type | Jira Issue | Notes |
|---|---|---|
| Epic | TBD | Suggest: "Move Money | Bill Pay_Recurring, Payee Management & Presentment (Part 3)" |
| Story / Task | TBD | To be created once this PRD is reviewed; see `BillPay_Action_Items.md` for items this PRD resolves (Action Items #1, #6) and creates new tracking needs for. |

---

## 13. Appendix

- Related PRDs: Part 1 — Send a Paper Check (`Vanteo_Connect_Bill_Pay_PRD_Part1_PaperCheck.docx`); Part 2 — Pay a Biller / ACH (`Vanteo_Connect_Bill_Pay_PRD_Part2_ElectronicBiller.docx`)
- Traceability references: `GOSPDR_BillPay_API_Inventory.md`, `BillPay_Traceability_Summary.md`, `BillPay_Screen_Traceability.md`, `BillPay_Action_Items.md`
- Get Scheduled Payments, Remove Biller, and Cancel Payment reuse endpoints already documented in Part 1's Appendix (SPDR API Docs) — no new vendor documentation needed for those three.
- Bill presentment and electronic biller removal/modification endpoints require new SPDR documentation or direct confirmation — none found in `docs.gospidr.com` references collected to date.
