# PRD Overview

By Christina Zikeli

## Document Control

**Feature Name:** Bill Pay — Send a Paper Check
**Product Area:** Move Money
**Target Release:** Phase 1

### Change Log

| Version | Date | Author | Summary |
|---|---|---|---|
| 1.0 | June 10, 2026 | Christina Zikeli | Initial draft — paper check, one-time payment |
| 1.1 | June 20, 2026 | Christina Zikeli | Added Landing Page UI spec, Use Case, Functional Requirements, Navigation |
| 1.2 | June 25, 2026 | Christina Zikeli | Moved Payment History to Phase 2; removed Screen Requirements section |
| 1.3 | June 30, 2026 | Christina Zikeli | Added Edit Payee, Review screens, UI Screen Inventory, versioning data model |

---

## 1. Overview

Bill Pay — Send a Paper Check is a Move Money feature within Vanteo Connect that allows a customer to send a one-time paper check to any business or individual (Pay Now). Vanteo prints and mails a physical check on the customer's behalf, debiting their designated DDA account. Phase 1 covers five screens: Bill Pay Landing Page, List of Payees, Payee Details, Add a New Payee, and Make a Payment, plus supporting screens for review, edit, and payment confirmation.

### In Scope

| Area | Description |
|---|---|
| Bill Pay Landing Page | Entry point showing saved payees, tab navigation, and Add a New Payee action |
| List of Payees | Paginated, searchable list of all saved paper check payees |
| Payee Details | Full payee info; gateway to Pay and Edit actions |
| Add a New Payee | Form to enter payee name, address, and optional phone number |
| Review Payee Details | Read-only confirmation of entered payee info before saving |
| Edit a Payee | Pre-populated form to update payee info; creates new versioned record |
| Create a Payment | Enter payment amount; confirm DDA account and balance |
| Review & Confirm Payment | Read-only payment summary before submitting |
| Payment Success | Confirmation of submitted payment with delivery notice |
| Save Payee to account | Persisting payee record via addPaperBiller API |
| DDA account display | Last 4 digits and available balance on payment screen |
| Cancel and back navigation | Exit any flow and return to Landing Page |

### Out of Scope

| Area | Description |
|---|---|
| View Payment History | List of past payments — Phase 2 |
| Schedule a future payment | Date-selected payments — Phase 2 |
| Recurring payments | Automatic repeating payments — Phase 2 |
| Delete a saved payee | Permanent payee removal — Phase 3 |
| Cancel a submitted payment | Reversing a sent check — Phase 2 |
| Electronic (ACH) biller payments | Direct biller payments from RPPS registry — Part 2 PRD |

---

## 2. Use Case

Jane is a foreign national on a temporary work assignment in the United States. She rents an apartment with a friend for five months. The monthly rent is $1,000, split evenly between them — Jane owes her roommate $500 each month, who then pays the landlord the full $1,000.

Jane opens Bill Pay in Vanteo Connect, taps Add a New Payee, and enters her roommate's mailing address. She reviews the payee details on the Review Payee Details screen, confirms, and is taken to Create a Payment. She enters $500, reviews the payment on the Review & Confirm Payment screen, taps Submit, and sees the Payment Success screen. Vanteo deducts $500 from Jane's DDA account and mails a physical paper check.

The following month, Jane's roommate moves. Jane returns to Bill Pay, opens Payee Details, and taps Edit. She updates the mailing address. The system creates a new payee record with a new Payee ID, records the date of change, and preserves the old record. All prior payments remain linked to the old address. Future payments use the updated record.

### Key Takeaways

| Insight | PRD Implication |
|---|---|
| User sends money to an individual, not just a business | Payee Name field accepts any name — person or business |
| Timing matters: check must arrive before the 15th | Pay Now sends today; mailing lead-time message is critical |
| User returns monthly to repeat the payment | List of Payees and Payee Details reduce friction for repeat payments |
| Payee address can change between payments | Edit Payee creates a new versioned record; old address preserved for payment history accuracy |
| User is a foreign national less familiar with US bill pay | How Bill Pay Works guidance and info banner are important orientation tools |
| Funds deducted from a specific DDA account | Make a Payment shows last 4 digits and available balance to confirm correct account |

---

## 3. Assumptions

- Primary users are foreign nationals on temporary work or student visas living in the United States.
- Each user holds at least one active Vanteo DDA account at the time of using Bill Pay.
- Payees are US-based; paper checks are mailed via US Postal Service to domestic addresses only.
- All Phase 1 payments are one-time (Pay Now). Scheduling and recurring payments are not supported.
- The user has completed onboarding and identity verification before accessing Bill Pay.
- Paper check delivery lead time is 7–10 business days (to be confirmed with Ops).
- The SPDR API (spidrsrv.com) is the backend powering all bill pay operations.
- A single DDA account is used per payment; multi-account selection is out of scope for Phase 1.

---

## 4. UI Screen Inventory

| # | Screen Name | Purpose | Entry Point |
|---|---|---|---|
| 1 | Bill Pay Landing Page | Entry point; shows saved payees and tab navigation | Move Money hub |
| 2 | List of Payees | Paginated, searchable list of all saved payees | Landing Page — Payees tab |
| 3 | Payee Details | Full payee info; gateway to Pay and Edit actions | Landing Page (payee row) or List of Payees (payee row) |
| 4 | Add a New Payee | Form to enter name, address, and optional phone number | Landing Page — Add a New Payee row |
| 5 | Review Payee Details | Read-only summary of entered payee info before saving | Add a New Payee — Continue button |
| 6 | Edit a Payee | Pre-populated form to update payee info; creates new versioned record | Payee Details — Edit button |
| 7 | Create a Payment | Enter payment amount; confirm DDA account and balance | Review Payee Details (new payee) or Payee Details — Pay button (saved payee) |
| 8 | Review & Confirm Payment | Read-only summary of payment before submitting | Create a Payment — Review button |
| 9 | Payment Success | Confirmation of submitted payment with delivery notice | Review & Confirm Payment — Submit button |

---

## 5. User Flow

### Primary Flow — New Payee Path

| Step | User Action | System Response |
|---|---|---|
| 1 | Open Bill Pay from Move Money hub | Load Landing Page; call GET /billpay/getBillers |
| 2 | Tap Add a New Payee row | Navigate to Add a New Payee |
| 3 | Enter payee name and address; tap Continue | Validate required fields; navigate to Review Payee Details |
| 4 | Review payee info; tap Save & Pay | Call POST /billpay/addPaperBiller; on success navigate to Create a Payment |
| 5 | Confirm DDA account and balance; enter amount; tap Review | Validate amount; navigate to Review & Confirm Payment |
| 6 | Review payment summary; tap Submit | Call POST /billpay/createPayment; on success navigate to Payment Success |
| 7 | Tap Done | Return to Landing Page; clear navigation stack |

### Alternate Flows

**Saved Payee Path**

| Step | User Action | System Response |
|---|---|---|
| 1 | Tap Payees tab on Landing Page | Navigate to List of Payees |
| 2 | Search or scroll; tap a payee row | Navigate to Payee Details |
| 3 | Tap Pay | Navigate to Create a Payment with payee pre-selected |
| 4 | Enter amount; tap Review | Navigate to Review & Confirm Payment |
| 5 | Tap Submit | Call POST /billpay/createPayment; navigate to Payment Success |
| 6 | Tap Done | Return to Landing Page |

**Edit Payee Path**

| Step | User Action | System Response |
|---|---|---|
| 1 | From Payee Details, tap Edit (pencil icon or address Edit button) | Navigate to Edit a Payee with all fields pre-populated |
| 2 | Update fields; tap Save Changes | Call POST /billpay/modifyPaperBiller; create new payee record with new Payee ID |
| 3 | Save confirmed | Navigate to Payee Details showing new record |

**Cancel Path (any screen)**

| Step | User Action | System Response |
|---|---|---|
| 1 | Tap Cancel from any screen in an active flow | Discard all unsaved data |
| 2 | — | Return to Bill Pay Landing Page |

---

## 6. Functional Requirements

### Bill Pay Landing Page

| ID | Requirement | Priority |
|---|---|---|
| FR-01 | App shall call GET /billpay/getBillers on page load and display all saved paper check payees. | High |
| FR-02 | App shall display a payee count badge with the total number of saved payees. | Medium |
| FR-03 | App shall provide a real-time search bar filtering the payee list as the user types. | Medium |
| FR-04 | Each payee row shall display: payee name, last-paid date, and a right chevron. | High |
| FR-05 | App shall display an Add a New Payee row with dashed top border and subtext at the bottom of the payee list. | High |
| FR-06 | App shall show an empty state with only the Add a New Payee row when no payees are saved. | High |
| FR-07 | The Payees tab shall navigate to the List of Payees screen. | High |
| FR-08 | The History, Billers, and Upcoming tabs shall be visible but inactive (greyed out) in Phase 1. | Medium |

### List of Payees

| ID | Requirement | Priority |
|---|---|---|
| FR-09 | App shall display a paginated list of saved paper check payees, 25 per initial page. | High |
| FR-10 | App shall display a search bar filtering by payee name, city, or state in real time; a new search resets to page 1. | High |
| FR-11 | Each payee row shall display: payee name and city/state only. | High |
| FR-12 | App shall display column headers: Payee Name and City, State. | Medium |
| FR-13 | App shall display a result count footer: Showing X of Y payees. | Medium |
| FR-14 | App shall display a View More Payees button when total results exceed the current page; button hides when all results are visible. | High |
| FR-15 | Tapping View More Payees shall load the next 20 payees and append them to the list. | High |
| FR-16 | A dismissable blue info banner shall appear above the list explaining paper check delivery via US Postal Service. | Medium |
| FR-17 | Tapping a payee row shall navigate to the Payee Details screen for that payee. | High |

### Payee Details

| ID | Requirement | Priority |
|---|---|---|
| FR-18 | App shall display full payee information: payee name, street address, address line 2 (if present), city, state, ZIP, and phone number (if present). | High |
| FR-19 | App shall display an Edit button (pencil icon) in the hero top-right that navigates to Edit a Payee. | High |
| FR-20 | App shall display a Delete button (red trash icon) in the hero top-right. Visible in Phase 1; inactive — Delete is Phase 3. | Medium |
| FR-21 | The Payee Address section shall include an Edit button in its header that also navigates to Edit a Payee. | High |
| FR-22 | App shall display a Pay button that navigates to Create a Payment with this payee pre-selected. | High |
| FR-23 | After an edit is saved, Payee Details shall reload and display the newly created payee record. | High |

### Add a New Payee

| ID | Requirement | Priority |
|---|---|---|
| FR-24 | App shall present: Payee Name, Address Line 1, Address Line 2, City, State, ZIP Code, Phone Number. | High |
| FR-25 | Payee Name, Address Line 1, City, State, and ZIP Code are required fields. | High |
| FR-26 | Phone Number and Address Line 2 are optional. | High |
| FR-27 | Continue button shall be disabled until all required fields contain valid input. | High |
| FR-28 | State shall be a dropdown limited to US states and territories. | High |
| FR-29 | ZIP Code shall accept exactly 5 numeric digits. | High |
| FR-30 | Tapping Continue navigates to Review Payee Details. No API call is made at this step. | High |
| FR-31 | Cancel or back navigation shall discard unsaved data and return to Landing Page. | High |

### Review Payee Details

| ID | Requirement | Priority |
|---|---|---|
| FR-32 | App shall display a read-only summary of all payee fields entered on Add a New Payee. | High |
| FR-33 | App shall provide a back/edit action that returns to Add a New Payee with all fields pre-populated for correction. | High |
| FR-34 | Save & Pay button shall call POST /billpay/addPaperBiller. On success, navigate to Create a Payment. | High |
| FR-35 | On API failure, app shall display an inline error banner and allow retry without leaving the screen. | High |
| FR-36 | Cancel shall discard all unsaved payee data and return to Landing Page. | High |

### Edit a Payee

| ID | Requirement | Priority |
|---|---|---|
| FR-37 | App shall pre-populate all fields with the current payee record values. | High |
| FR-38 | Required field rules are identical to Add a New Payee. Save Changes button is disabled until all required fields are valid. | High |
| FR-39 | On Save Changes, app shall call POST /billpay/modifyPaperBiller with the updated fields. | High |
| FR-40 | The API shall create a new payee record with a new Payee ID. The system shall record the date of change (modifiedDate) on the new record. | High |
| FR-41 | The original payee record shall not be deleted or overwritten. It shall remain in the system linked to any prior payments. | High |
| FR-42 | All payments made before the edit shall remain associated with the old Payee ID and old payee address. | High |
| FR-43 | After a successful save, app shall navigate to Payee Details displaying the new payee record. | High |
| FR-44 | On API failure, app shall display an inline error and allow retry. | High |
| FR-45 | Cancel shall discard all edits and return to Payee Details with no changes made. | High |

### Create a Payment

| ID | Requirement | Priority |
|---|---|---|
| FR-46 | App shall display the selected payee name at the top (read-only). | High |
| FR-47 | App shall display the DDA account with last 4 digits masked (e.g., Checking 1234). | High |
| FR-48 | App shall display the current available account balance. | High |
| FR-49 | App shall provide an Amount field: numeric keyboard, 2 decimal places, non-zero required. | High |
| FR-50 | Payment Frequency shall display as a read-only label: One-Time Payment, Pay Now. | High |
| FR-51 | Review button shall be disabled until a valid amount is entered. | High |
| FR-52 | Tapping Review navigates to Review & Confirm Payment. No API call is made at this step. | High |
| FR-53 | Cancel shall return the user to the Landing Page without submitting a payment. | High |

### Review & Confirm Payment

| ID | Requirement | Priority |
|---|---|---|
| FR-54 | App shall display a read-only payment summary: payee name, amount, DDA account (masked), payment method (Paper Check), payment date (today). | High |
| FR-55 | App shall display a notice: Allow 7–10 business days for your check to arrive. | High |
| FR-56 | App shall provide a back action that returns to Create a Payment with the amount pre-filled. | High |
| FR-57 | Submit button shall call POST /billpay/createPayment with processDate = today and the active Payee ID. | High |
| FR-58 | On success, navigate to Payment Success. On failure, display inline error with retry. | High |
| FR-59 | Cancel shall return to Landing Page. Payment is not submitted. | High |

### Payment Success

| ID | Requirement | Priority |
|---|---|---|
| FR-60 | App shall display success state: payee name, amount, payment method (Paper Check). | High |
| FR-61 | App shall display delivery notice: Allow 7–10 business days for your check to arrive. | High |
| FR-62 | Done button shall return to the Landing Page. Back navigation is disabled on this screen. | High |

---

## 7. Data Model / Versioning

When a user edits a payee, the system creates a new versioned record rather than overwriting the existing one. This preserves historical payment accuracy.

| Field / Object | Current State | New State | Notes |
|---|---|---|---|
| Payee ID | Original ID (e.g., BLR-1001) | New ID (e.g., BLR-1042) generated on edit | Old ID retained; new ID used for future payments |
| Payee Name | Original value | Updated value (or same if unchanged) | Stored on new record |
| Payee Address | Original address | Updated address | Old address preserved on original record |
| Phone Number | Original value | Updated value (or same if unchanged) | Optional field |
| Created Date | Original creation date | Date of edit (modifiedDate) | Recorded on new record |
| Record Status | Active | New record = Active; original = Inactive (superseded) | Only Active records returned by GET /billpay/getBillers |
| Linked Payments | All payments made before the edit | All payments made after the edit | Historical payments retain old Payee ID reference |

---

## 8. Mobile App Navigation

### Navigation Rules

- **Back arrow** follows per-screen rules (see Navigation Map); it does not always return to Landing Page.
- **Cancel** always discards the active flow and returns to Bill Pay Landing Page, regardless of depth.
- **Done** on Payment Success returns to Landing Page and clears the navigation stack.
- **Back navigation is disabled** on Payment Success; the user must tap Done.
- Tapping a saved payee row on the Landing Page navigates directly to Payee Details, bypassing List of Payees.

### Navigation Map

| Trigger | From Screen | To Screen |
|---|---|---|
| Back arrow | Landing Page | Move Money |
| Payees tab | Landing Page | List of Payees |
| Payee row tap | Landing Page | Payee Details |
| Add a New Payee row tap | Landing Page | Add a New Payee |
| Back arrow | List of Payees | Landing Page |
| Payee row tap | List of Payees | Payee Details |
| View More Payees | List of Payees | Same screen (appends results) |
| Back arrow | Payee Details | Landing Page |
| Pay button | Payee Details | Create a Payment |
| Edit button (hero or address) | Payee Details | Edit a Payee |
| Continue button | Add a New Payee | Review Payee Details |
| Cancel / back arrow | Add a New Payee | Landing Page |
| Back arrow | Review Payee Details | Add a New Payee (fields pre-populated) |
| Save & Pay (success) | Review Payee Details | Create a Payment |
| Cancel | Review Payee Details | Landing Page |
| Save Changes (success) | Edit a Payee | Payee Details (new record) |
| Cancel / back arrow | Edit a Payee | Payee Details (no change) |
| Review button | Create a Payment | Review & Confirm Payment |
| Cancel | Create a Payment | Landing Page |
| Back arrow | Create a Payment | Payee Details or Review Payee Details |
| Submit (success) | Review & Confirm Payment | Payment Success |
| Back arrow | Review & Confirm Payment | Create a Payment |
| Cancel | Review & Confirm Payment | Landing Page |
| Done button | Payment Success | Landing Page (stack cleared) |

---

## 9. API Reference

| Method | Endpoint | Trigger | Key Parameters | Notes |
|---|---|---|---|---|
| GET | /billpay/getBillers | Landing Page on load; List of Payees | accountId | Returns Active payee records only. Superseded records not returned. |
| POST | /billpay/addPaperBiller | Save & Pay on Review Payee Details | accountId, billerName, billerCity, billerState, billerZip (required); billerAddress1, billerAddress2, billerPhone (optional); frequencyType = one_time | Returns billerId for immediate use in createPayment. |
| POST | /billpay/modifyPaperBiller | Save Changes on Edit a Payee | accountId, billerId (existing), billerName, billerCity, billerState, billerZip (required); billerAddress1, billerAddress2, billerPhone (optional) | Creates new record with new billerId; records modifiedDate. Does not delete original record. |
| POST | /billpay/createPayment | Submit on Review & Confirm Payment | accountId, billerId (active), amount, processDate = today (YYYY-MM-DD); memo (optional, max 50 chars) | Always use the active billerId. Historical payments retain original billerId. |

Base URL: `https://api.sb.spidrsrv.com/v1`

---

## 10. Error States

| Scenario | User Message | Action |
|---|---|---|
| GET /billpay/getBillers fails on load | Unable to load your payees. Pull to refresh. | Pull-to-refresh on Landing Page |
| POST /billpay/addPaperBiller fails | We couldn't save this payee. Please try again. | Retry button on Review Payee Details |
| POST /billpay/modifyPaperBiller fails | We couldn't update this payee. Please try again. | Retry button on Edit a Payee |
| POST /billpay/createPayment fails | Payment could not be submitted. Please try again. | Retry button on Review & Confirm Payment |
| Amount is zero or blank | Please enter a valid payment amount. | Inline field error on Create a Payment |
| Required payee field missing | Field-level highlight with label: Required | Inline field error on Add / Edit Payee |
| Network error (any screen) | Check your connection and try again. | Retry button |

---

## 11. Open Questions

| # | Question | Owner | Status |
|---|---|---|---|
| 1 | What account number value is sent to addPaperBiller? Biller-assigned, DDA, or something else? | Product / Engineering | Open |
| 2 | Confirmed paper check mailing lead time? (Assumed 7–10 business days.) | Ops | Open |
| 3 | Is ZTM enabled? If so, sessionKey required on addPaperBiller, modifyPaperBiller, and createPayment. | Engineering | Open |
| 4 | Does modifyPaperBiller return the new billerId synchronously? Does it set the original record to Inactive? | Engineering | Open |
| 5 | How is Account Balance fetched for Create a Payment? Existing account endpoint or separate call? | Engineering | Open |
| 6 | Should Billers, History, and Upcoming tabs be fully hidden or greyed out with a coming-soon tooltip in Phase 1? | Product / Design | Open |
| 7 | Should dismissing the info banner on List of Payees persist across sessions or reset each visit? | Product / Engineering | Open |
| 8 | Does List of Payees use GET /billpay/getBillers with client-side pagination, or a separate paginated API endpoint? | Engineering | Open |
| 9 | In Phase 1, should Delete on Payee Details be hidden, greyed out, or shown with a coming-soon tooltip? | Product / Design | Open |
| 10 | When a payee is edited and a new record created, confirm API filtering — only Active records returned by getBillers? | Engineering | Open |
| 11 | Should Edit a Payee be a full-form edit or allow inline editing of individual fields (e.g., address only)? | Product / Design | Open |

---

## 12. Jira Links

| Type | Jira Issue | Notes |
|---|---|---|
| Epic | TBD | Bill Pay — Send a Paper Check (Phase 1) |
| Story / Task | TBD | Link to implementation stories when created |

---

## 13. Appendix

- [Get Billers — SPDR API Docs](https://docs.gospidr.com/reference/getv1billpaygetbillers)
- [Add Paper Biller — SPDR API Docs](https://docs.gospidr.com/reference/postv1billpayaddpaperbiller)
- [Modify Paper Biller — SPDR API Docs](https://docs.gospidr.com/reference/postv1billpaymodifypaperbiller)
- [Create Bill Payment — SPDR API Docs](https://docs.gospidr.com/reference/postv1billpaycreatepayment)
