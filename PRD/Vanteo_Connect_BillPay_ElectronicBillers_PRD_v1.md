# PRD Overview

By Christina Zikeli

## Document Control

**Feature Name:** Bill Pay — Electronic Biller Payments (ACH)
**Product Area:** Move Money
**Target Release:** Part 2 — Electronic Billers

### Change Log

| Version | Date | Author | Summary |
|---|---|---|---|
| 1.0 | July 9, 2026 | Christina Zikeli | Initial draft — electronic biller ACH payments |

---

## 1. Overview

This PRD defines Part 2 of Bill Pay within Vanteo Connect: the ability for a customer to pay registered electronic billers via ACH. Billers must be registered in the RPPS biller directory. The customer searches and selects a biller, links their account number to the biller, and submits a one-time ACH payment. Payments post within 1–3 business days.

This document also introduces two new cards to the Bill Pay Landing Page: an Electronic Billers card showing the customer's active linked billers, and a Payment Due Coming Soon card surfacing upcoming payment due dates from linked billers.

### In Scope

| Area | Description |
|---|---|
| Bill Pay Landing Page — Electronic Billers Card | Displays up to 3 active linked billers; search bar; Add a New Biller link |
| Bill Pay Landing Page — Payment Due Coming Soon Card | Shows upcoming payment due info from linked billers |
| Search & Select a Biller | Search RPPS biller directory; default list starts with A; 10 per page |
| Biller Details | View biller name, address, and phone number |
| Link Invoice to Biller | Customer enters account number to link their account to the biller |
| Make a Payment | Enter payment amount against DDA account |
| Review & Confirm Payment | Read-only payment summary before submitting ACH |
| Payment Success | Confirmation with ACH posting timeline |
| View All Billers (Billers Tab) | Full list of billers linked in past 10 years; empty state handling |
| One-time ACH payment | Single payment only; 1–3 business day posting |

### Out of Scope

| Area | Description |
|---|---|
| Recurring / scheduled ACH payments | Future phase |
| Cancel a submitted ACH payment | Future phase |
| Edit a linked biller account number | Future phase |
| Remove / unlink a biller | Future phase |
| Paper check payments | Covered in Part 1 PRD |
| Payment History | Future phase |
| Multiple DDA account selection | Future phase |

---

## 2. Use Case

Marco is a foreign national on a work visa in the United States. Each month he receives bills from Duke Energy, Verizon, and Piedmont Gas. Each biller sends him a paper statement with a due date and account number.

Marco opens Bill Pay and wants to pay his monthly bills through his mobile app. He sees the Electronic Billers card on the Landing Page. He has not yet linked any billers, so only the Add a New Biller link is visible. He taps it, searches for Verizon by name, and selects it from the list. He views the Biller Details — name, address, and phone — and confirms they match his Verizon paper invoice. He proceeds to add his Verizon account number in Bill Pay and enters his 10-digit account number from his paper statement. His Verizon biller is now linked.

Marco taps Make a Payment, confirms his DDA account and available balance, enters $85.00, and reviews the payment summary. He taps Submit. The app confirms the payment was submitted via ACH and will post in 1–3 business days.

The following month, Marco returns to Bill Pay. The Electronic Billers card now shows Verizon with a search bar. The Payment Due card shows his upcoming Verizon bill: due date, amount owed, and account number. Marco selects Verizon and pays the bill in seconds.

### Key Takeaways

| Insight | PRD Implication |
|---|---|
| Billers must be registered in the RPPS directory | Search must query the RPPS biller directory only |
| Customer has a unique account number per biller | Link Invoice step must capture and store the account number |
| Foreign nationals may be unfamiliar with ACH bill pay | Clear posting timeline (1–3 business days) must be displayed at review and confirmation |
| Customer has multiple billers | Billers tab and Landing Page card reduce friction for managing multiple billers |
| Payment due dates come from the biller | Payment Due Coming Soon card surfaces biller-provided due date data |
| Customer pays the exact amount on the statement | Amount field should not be pre-filled; customer enters the amount from their statement |

---

## 3. Assumptions

- Electronic billers must be registered in the RPPS biller directory to appear in search results.
- Each customer has one active Vanteo DDA account used for ACH payments in Phase 1.
- One-time payments only; recurring and scheduled payments are out of scope.
- ACH payment posting timeline is 1–3 business days (standard ACH processing).
- The customer has a biller-issued account number required for the Link Invoice step.
- Payment Due Coming Soon data is sourced from the biller via the SPDR API; availability depends on biller participation.
- The user has completed identity verification and onboarding before accessing Bill Pay.
- The Billers tab displays billers linked within the past 10 years, regardless of current activity.
- Search results default to billers starting with the letter A when the search bar is first opened, 10 per page.

---

## 4. UI Screen Inventory

| # | Screen Name | Purpose | Entry Point |
|---|---|---|---|
| 1 | Bill Pay Landing Page (updated) | Entry point; Electronic Billers card + Payment Due Coming Soon card added | Move Money hub |
| 2 | Search & Select a Biller | Search RPPS biller directory; select a biller to proceed | Landing Page — Add a New Biller; Billers tab — Add a New Biller |
| 3 | Biller Details | View biller name, address, and phone number | Search & Select — biller row tap |
| 4 | Link Invoice to Biller | Enter customer account number to link to the biller | Biller Details — Link Invoice button |
| 5 | Make a Payment | Enter payment amount; confirm DDA account and balance | Linked Biller — Pay button; Billers tab — Pay button |
| 6 | Review & Confirm Payment | Read-only payment summary before ACH submission | Make a Payment — Review button |
| 7 | Payment Success | ACH payment submitted confirmation with posting timeline | Review & Confirm — Submit button |
| 8 | View All Billers | Full list of billers linked in past 10 years; empty state | Billers tab on Landing Page |

---

## 5. User Flow

### Primary Flow — New Biller + Payment

| Step | User Action | System Response |
|---|---|---|
| 1 | Open Bill Pay from Move Money hub | Load Landing Page; call GET /billpay/getBillers; show Electronic Billers card and Payment Due Coming Soon card |
| 2 | Tap Add a New Biller on Electronic Billers card | Navigate to Search & Select a Biller; call GET /billpay/searchBillerDirectory with default query (A); display first 10 results |
| 3 | Type biller name in search bar | Call GET /billpay/searchBillerDirectory with search term; display up to 10 matching billers |
| 4 | Tap a biller row | Navigate to Biller Details; display biller name, address, phone |
| 5 | Tap Link Invoice | Navigate to Link Invoice to Biller |
| 6 | Enter account number; tap Save & Continue | Validate account number; call link biller API; on success navigate to Make a Payment |
| 7 | Review DDA account and balance; enter amount; tap Review | Validate amount; navigate to Review & Confirm Payment |
| 8 | Review payment summary; tap Submit | Call POST /billpay/createPayment (ACH); on success navigate to Payment Success |
| 9 | Tap Done | Return to Bill Pay Landing Page; stack cleared |

### Alternate Flows

**Returning Biller Path**

| Step | User Action | System Response |
|---|---|---|
| 1 | Open Bill Pay Landing Page | Electronic Billers card shows up to 3 active linked billers with search bar |
| 2 | Tap a biller row on the card | Navigate to Make a Payment with biller pre-selected |
| 3 | Enter amount; tap Review | Navigate to Review & Confirm Payment |
| 4 | Tap Submit | Call POST /billpay/createPayment; navigate to Payment Success |
| 5 | Tap Done | Return to Landing Page |

**View All Billers Path**

| Step | User Action | System Response |
|---|---|---|
| 1 | Tap Billers tab on Landing Page | Navigate to View All Billers; call GET /billpay/getBillers (all, 10-year history) |
| 2 | Browse or search biller list | Filter list in real time |
| 3 | Tap a biller row | Navigate to Make a Payment with biller pre-selected |
| 4 | Tap Add a New Biller | Navigate to Search & Select a Biller |

**Cancel Path (any screen)**

| Step | User Action | System Response |
|---|---|---|
| 1 | Tap Cancel from any screen | Discard unsaved data |
| 2 | — | Return to Bill Pay Landing Page |

---

## 6. Functional Requirements

### Bill Pay Landing Page — Electronic Billers Card

| ID | Requirement | Priority |
|---|---|---|
| EB-01 | App shall display an Electronic Billers card on the Bill Pay Landing Page below the Paper Check Payees card. | High |
| EB-02 | The Electronic Billers card shall call GET /billpay/getBillers on page load and display up to 3 active linked electronic billers. | High |
| EB-03 | Each biller row on the card shall display the biller name and a right chevron. | High |
| EB-04 | The card shall display a search bar above the biller list when at least 1 active biller is linked. | High |
| EB-05 | The search bar shall filter the card's biller list in real time as the user types. | Medium |
| EB-06 | The card shall display an Add a New Biller link at the bottom. The link is always visible regardless of biller count. | High |
| EB-07 | If no billers are active, the card shall display only the Add a New Biller link. No search bar and no biller rows are shown. | High |
| EB-08 | Tapping a biller row on the card shall navigate to Make a Payment with that biller pre-selected. | High |
| EB-09 | Tapping Add a New Biller shall navigate to Search & Select a Biller. | High |

### Bill Pay Landing Page — Payment Due Coming Soon Card

| ID | Requirement | Priority |
|---|---|---|
| EB-10 | App shall display a Payment Due Coming Soon card on the Bill Pay Landing Page when upcoming payment due data is available from linked billers. | High |
| EB-11 | Each item in the card shall display: biller name, payment due date, payment amount due, and customer account number. | High |
| EB-12 | The card shall display the customer's name associated with each biller account. | High |
| EB-13 | If no upcoming payment data is available from any linked biller, the Payment Due Coming Soon card shall be hidden. | Medium |
| EB-14 | Tapping a payment due item shall navigate to Make a Payment with the biller pre-selected and amount pre-filled if the biller provides the due amount. | Medium |

### Search & Select a Biller

| ID | Requirement | Priority |
|---|---|---|
| EB-15 | App shall display a search bar at the top of the screen. | High |
| EB-16 | On screen load, app shall call GET /billpay/searchBillerDirectory with a default query returning billers that start with the letter A. | High |
| EB-17 | App shall display up to 10 billers per page. | High |
| EB-18 | As the user types in the search bar, app shall call GET /billpay/searchBillerDirectory with the typed term and display up to 10 matching results. | High |
| EB-19 | Each biller row shall display the biller name. | High |
| EB-20 | App shall display a Load More button when additional results are available beyond the current 10. | Medium |
| EB-21 | Tapping a biller row shall navigate to Biller Details for the selected biller. | High |
| EB-22 | If no results match the search term, app shall display a no-results state. | High |
| EB-23 | Back arrow shall return to the previous screen (Landing Page or View All Billers). | High |

### Biller Details

| ID | Requirement | Priority |
|---|---|---|
| EB-24 | App shall display the selected biller's name, full mailing address, and phone number. | High |
| EB-25 | All information is read-only and sourced from the RPPS biller directory. | High |
| EB-26 | App shall display a Link Invoice button to proceed to account linking. | High |
| EB-27 | Back arrow shall return to Search & Select a Biller. | High |

### Link Invoice to Biller

| ID | Requirement | Priority |
|---|---|---|
| EB-28 | App shall display the selected biller name (read-only) and an Account Number input field. | High |
| EB-29 | The Account Number field shall be required. Save & Continue button is disabled until a value is entered. | High |
| EB-30 | On Save & Continue, app shall call the biller link API to associate the customer's account number with the selected biller. | High |
| EB-31 | On success, app shall navigate to Make a Payment with the biller and account number pre-selected. | High |
| EB-32 | On API failure, app shall display an inline error and allow retry without leaving the screen. | High |
| EB-33 | Cancel shall return to Bill Pay Landing Page; no biller is linked. | High |

### Make a Payment

| ID | Requirement | Priority |
|---|---|---|
| EB-34 | App shall display the selected biller name (read-only). | High |
| EB-35 | App shall display the DDA account with last 4 digits masked (e.g., Checking 1234). | High |
| EB-36 | App shall display the current available account balance. | High |
| EB-37 | App shall provide an Amount field: numeric keyboard, 2 decimal places, non-zero required. | High |
| EB-38 | App shall display Payment Method as a read-only label: ACH — Electronic Payment. | High |
| EB-39 | App shall display posting notice: Payments take 1–3 business days to post. | High |
| EB-40 | Review button shall be disabled until a valid amount is entered. | High |
| EB-41 | Tapping Review navigates to Review & Confirm Payment. No API call is made at this step. | High |
| EB-42 | Cancel shall return to Landing Page without submitting a payment. | High |

### Review & Confirm Payment

| ID | Requirement | Priority |
|---|---|---|
| EB-43 | App shall display a read-only payment summary: biller name, payment amount, DDA account (masked), payment method (ACH — Electronic Payment), and payment date (today). | High |
| EB-44 | App shall display a notice: ACH payments take 1–3 business days to post to your account. | High |
| EB-45 | App shall provide a back action that returns to Make a Payment with amount pre-filled. | High |
| EB-46 | Submit button shall call POST /billpay/createPayment with payment type = ACH and processDate = today. | High |
| EB-47 | On success, navigate to Payment Success. On failure, display inline error with retry. | High |
| EB-48 | Cancel shall return to Landing Page. No payment is submitted. | High |

### Payment Success

| ID | Requirement | Priority |
|---|---|---|
| EB-49 | App shall display success state: biller name, amount, payment method (ACH — Electronic Payment). | High |
| EB-50 | App shall display posting notice: Your payment will post in 1–3 business days. | High |
| EB-51 | Done button shall return to the Landing Page. Back navigation is disabled on this screen. | High |

### View All Billers (Billers Tab)

| ID | Requirement | Priority |
|---|---|---|
| EB-52 | Tapping the Billers tab on the Landing Page shall navigate to the View All Billers screen. | High |
| EB-53 | App shall call GET /billpay/getBillers and display all billers linked to the customer in the past 10 years. | High |
| EB-54 | Each biller row shall display: biller name and a right chevron. | High |
| EB-55 | If at least 1 biller is listed, a search bar shall appear above the list. The search bar filters in real time. | High |
| EB-56 | If no billers are linked in the past 10 years, app shall display an empty state with no search bar and no biller rows. | High |
| EB-57 | App shall display an Add a New Biller link in the empty state and below the biller list. | High |
| EB-58 | Tapping a biller row shall navigate to Make a Payment with that biller pre-selected. | High |
| EB-59 | Tapping Add a New Biller shall navigate to Search & Select a Biller. | High |
| EB-60 | Back arrow shall return to Bill Pay Landing Page. | High |

---

## 7. Data Model / Versioning

| Field / Object | Current State | New State | Notes |
|---|---|---|---|
| Biller record | Paper check payee (name, address) | Electronic biller (RPPS biller ID, name, address, phone) | Separate record type from paper check payee |
| Customer–Biller Link | Not applicable | Account number linking customer to a specific biller | Created at Link Invoice step; stored per biller |
| Payment type | Paper check | ACH (electronic) | Payment type flag distinguishes paper vs. ACH in createPayment |
| Biller history window | Not applicable | Billers linked within past 10 years | Used for View All Billers screen |
| Payment Due data | Not applicable | Biller-provided due date, amount, customer account | Sourced from SPDR/biller; availability depends on biller participation |

---

## 8. Mobile App Navigation

### Navigation Rules

- **Back arrow** follows per-screen rules (see Navigation Map).
- **Cancel** on any screen in an active flow discards unsaved data and returns to Bill Pay Landing Page.
- **Done** on Payment Success returns to Landing Page and clears the navigation stack.
- **Back navigation is disabled** on Payment Success; the user must tap Done.
- Tapping a biller row on the Electronic Billers card navigates directly to Make a Payment, bypassing Biller Details and Link Invoice (biller is already linked).

### Navigation Map

| Trigger | From Screen | To Screen |
|---|---|---|
| Back arrow | Landing Page | Move Money |
| Billers tab | Landing Page | View All Billers |
| Biller row tap | Electronic Billers card | Make a Payment |
| Add a New Biller | Electronic Billers card | Search & Select a Biller |
| Payment Due item tap | Payment Due Coming Soon card | Make a Payment (biller pre-selected) |
| Back arrow | View All Billers | Landing Page |
| Add a New Biller | View All Billers | Search & Select a Biller |
| Biller row tap | View All Billers | Make a Payment |
| Biller row tap | Search & Select a Biller | Biller Details |
| Back arrow | Search & Select a Biller | Landing Page or View All Billers |
| Link Invoice button | Biller Details | Link Invoice to Biller |
| Back arrow | Biller Details | Search & Select a Biller |
| Save & Continue (success) | Link Invoice to Biller | Make a Payment |
| Cancel / back arrow | Link Invoice to Biller | Landing Page |
| Review button | Make a Payment | Review & Confirm Payment |
| Cancel | Make a Payment | Landing Page |
| Back arrow | Make a Payment | Biller Details or View All Billers |
| Submit (success) | Review & Confirm Payment | Payment Success |
| Back arrow | Review & Confirm Payment | Make a Payment (amount pre-filled) |
| Cancel | Review & Confirm Payment | Landing Page |
| Done button | Payment Success | Landing Page (stack cleared) |

---

## 9. API Reference

| Method | Endpoint | Trigger | Key Parameters | Notes |
|---|---|---|---|---|
| GET | /billpay/getBillers | Landing Page on load; View All Billers | accountId | Returns linked billers. Filter by biller type = electronic for Electronic Billers card. 10-year history window for View All Billers. |
| GET | /billpay/searchBillerDirectory | Search & Select a Biller on load and on each search input | searchTerm (default = "A"), pageSize = 10, pageNumber | RPPS biller directory search. Returns biller name, address, phone, billerId. |
| POST | /billpay/linkBiller (TBD) | Save & Continue on Link Invoice to Biller | accountId, billerId, customerAccountNumber | Links customer account number to biller. Endpoint name and payload to be confirmed with Engineering. |
| POST | /billpay/createPayment | Submit on Review & Confirm Payment | accountId, billerId, amount, processDate = today (YYYY-MM-DD), paymentType = ACH | ACH payment creation. Same endpoint as paper check with paymentType flag. To be confirmed. |

Base URL: `https://api.sb.spidrsrv.com/v1`

---

## 10. Error States

| Scenario | User Message | Action |
|---|---|---|
| GET /billpay/getBillers fails on load | Unable to load your billers. Pull to refresh. | Pull-to-refresh on Landing Page |
| GET /billpay/searchBillerDirectory fails | Unable to search billers. Please try again. | Retry button on Search & Select |
| No results for search term | No billers found for "[term]". Try a different name. | Inline no-results state |
| Link biller API fails | We couldn't link this biller. Please try again. | Retry button on Link Invoice |
| Create Payment API fails | Payment could not be submitted. Please try again. | Retry button on Review & Confirm Payment |
| Amount is zero or blank | Please enter a valid payment amount. | Inline field error on Make a Payment |
| Account number field blank | Please enter your account number. | Inline field error on Link Invoice |
| Network error (any screen) | Check your connection and try again. | Retry button |

---

## 11. Open Questions

| # | Question | Owner | Status |
|---|---|---|---|
| 1 | What is the exact SPDR API endpoint for linking a customer account number to a biller? What is the request payload and response schema? | Engineering | Open |
| 2 | Does the RPPS biller directory search return biller address and phone, or only biller name and ID? | Engineering | Open |
| 3 | What is the source of Payment Due Coming Soon data? Does SPDR provide this, or does it require a separate biller data feed? | Engineering / Product | Open |
| 4 | How is the 10-year biller history window enforced — client-side filter or API parameter? | Engineering | Open |
| 5 | Should the Electronic Billers card on the Landing Page show the 3 most recently paid billers, or alphabetically sorted? | Product / Design | Open |
| 6 | Can a customer link the same biller multiple times with different account numbers (e.g., two separate electricity accounts)? | Product / Engineering | Open |
| 7 | Is account number validation required at the Link Invoice step, or is it free-text? Does the biller validate the account number? | Engineering | Open |
| 8 | Should the Billers tab show all linked billers (active and inactive) or only active? What defines "active" for electronic billers? | Product / Engineering | Open |
| 9 | What payment types does createPayment accept? Is paymentType = ACH the correct flag to distinguish from paper check? | Engineering | Open |
| 10 | Should the amount on Make a Payment be pre-filled from the Payment Due Coming Soon data, or always left blank for the user to enter? | Product | Open |
| 11 | Is ZTM / session authentication required on searchBillerDirectory, linkBiller, and createPayment? | Engineering | Open |

---

## 12. Jira Links

| Type | Jira Issue | Notes |
|---|---|---|
| Epic | TBD | Bill Pay — Electronic Biller Payments (ACH) |
| Story / Task | TBD | Link to implementation stories when created |

---

## 13. Appendix

- [Search Biller Directory — SPDR API Docs](https://docs.gospidr.com/reference/getv1billpaysearchbillerdirectory)
- [Get Billers — SPDR API Docs](https://docs.gospidr.com/reference/getv1billpaygetbillers)
- [Create Bill Payment — SPDR API Docs](https://docs.gospidr.com/reference/postv1billpaycreatepayment)
- [Bill Pay Part 1 PRD — Send a Paper Check](Vanteo_Connect_BillPay_PRD_v6.md)
