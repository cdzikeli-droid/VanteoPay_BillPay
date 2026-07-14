# GOSPDR Bill Pay API Inventory

**Product:** Vanteo Connect — Bill Pay
**Base URL (SPDR/GOSPDR):** `https://api.sb.spidrsrv.com/v1`
**Base URL (Vanteo internal API, wraps SPDR):** `/v1/billpay/...` (Vanteo's own backend, per Jira dev tasks)
**Source:** SPDR (Galileo Bill Pay platform, on behalf of sponsor bank FBOL) — [docs.gospidr.com](https://docs.gospidr.com)
**Jira source:** `JIRA_VC_Full_Epic_Report 7.14.2026.xlsx` (202 issues, project VC)
**Purpose:** Master reference of every GOSPDR Bill Pay endpoint referenced across Vanteo Connect PRDs, mapped to its Jira Epic/Story/Dev Task.
**Last compiled:** 2026-07-14

> **Architecture note:** Two API layers are in play. GOSPDR/SPDR endpoints (`/billpay/getBillers`, `/billpay/addPaperBiller`, etc.) are the external vendor APIs documented in the PRDs. Vanteo's backend wraps these in its own REST layer (`/v1/billpay/payees`, `/v1/billpay/billers`, `/v1/billpay/payments`) — that's the layer the Jira dev tasks actually implement. Both are shown below so traceability holds across PRD → Jira → code.

---

## 1. Paper Check APIs (Part 1 — Mail a Check)

**Jira Epic: VC-67 — "Move Money | Bill Pay_Send a Paper Check to a Payee"** (Status: To Do)

| # | API Name | SPDR Endpoint | Vanteo Internal API | Jira Story | Jira Dev Task(s) | Notes / Gaps |
|---|---|---|---|---|---|---|
| 1 | Get Billers | GET `/billpay/getBillers` | GET `/v1/billpay/payees` (list) + GET `/v1/billpay/payees/:payeeId` (detail) | VC-132 (Bill Pay Home Landing Page), VC-133 (List of Payees), VC-141 (Review Payee Details), VC-137 (Select/View Payee Details) | VC-148 (SpidrClient methods), VC-149 (BE list/detail endpoints), VC-155/VC-161/VC-173 (FE API integration) | Fully traced. |
| 2 | Add Paper Biller | POST `/billpay/addPaperBiller` | POST `/v1/billpay/payees` | VC-136 (Add a new Payee) | VC-148, VC-150, VC-157 (FE API integration) | Fully traced. |
| 3 | Modify Paper Biller | POST `/billpay/modifyPaperBiller` | PUT `/v1/billpay/payees/:payeeId` | VC-142 (Edit a Payee) | VC-148, VC-151, VC-169 (FE API integration) | Fully traced. |
| 4 | **Remove Biller** | POST `/billpay/removeBiller` | *(none found)* | **None found** | **None found** | **Gap — no Jira story or dev task exists for Remove Biller anywhere in the report.** PRD documents it (§4.5, Data Flows) but it isn't tracked in Jira at all. Needs a story + BE task before it can be built or considered in scope. |
| 5 | Create Bill Payment (one-time paper check) | POST `/billpay/createPayment` | POST `/v1/billpay/payments` | VC-138 (Create a Payment), VC-134 (Review and Confirm Payment), VC-140 (Payment Success) | VC-148 (`createBillPayment` SpidrClient method), VC-152 (BE: send a one-time paper check), VC-163/VC-165/VC-167 (FE API integration) | Fully traced. |
| 6 | Cancel Payment | POST `/billpay/cancelPayment` | *(none found)* | VC-195 ("Cancel Payment Flow") | **None found** | **Epic mismatch + partial gap.** PRD Part 1 scopes Cancel Payment under paper checks, but the only matching Jira story (VC-195) sits under Epic VC-184 (ACH Billers), not VC-67. No backend dev task implements the endpoint under either epic. |
| 7 | Get Payment History | GET `/billpay/getPaymentHistory` | GET `/v1/billpay/payments` (history) | VC-135 ("Display Bill Pay Payment History") | VC-205 (BE: payment history, live from Spidr) | **Epic mismatch.** PRD Part 1 scopes this under paper check, but story and dev task both live under Epic VC-184 (ACH Billers), not VC-67. Functionally traced, just cross-epic — confirm this is intentional (payment history may have been consolidated to cover both payment types). |
| 8 | Get Scheduled Payments | GET `/billpay/getScheduledPayments` | *(none found)* | VC-188 ("Bill Pay Home Page with Payment Due Cards") — under Epic VC-184 | VC-206 ("Spike: Payment Due Coming Soon data source (PRD OQ#3)") | **Still unresolved.** The only related Jira item is a *spike* (investigation), not a committed dev task — consistent with the PRD gap already flagged (no formal parameter spec exists for this endpoint). Don't consider this API ready for implementation. |

---

## 2. Electronic / ACH Biller APIs (Part 2 — Pay a Biller)

**Jira Epic: VC-184 — "Move Money | Bill Pay_ACH Billers"** (Status: To Do)

| # | API Name | SPDR Endpoint | Vanteo Internal API | Jira Story | Jira Dev Task(s) | Notes / Gaps |
|---|---|---|---|---|---|---|
| 9 | Get Billers (electronic filter) | GET `/billpay/getBillers?billerType=electronic` | GET `/v1/billpay/billers` (list linked electronic billers) | VC-187 (Bill Pay Home Page w Biller List Card), VC-196 (View List of Linked Billers) | VC-199 (Spidr client: electronic-biller & directory methods), VC-201 (BE: GET list linked electronic billers) | Fully traced. |
| 10 | Search Billers (RPPS directory) | GET `/billpay/searchBillers` | GET `/v1/billpay/billers/directory` | VC-189 (Search and Select a Biller), VC-190 (View Biller Details) | VC-199, VC-202 (BE: search RPPS biller directory) | Fully traced. Confirms draft name `searchBillerDirectory` (see §3) was renamed to `searchBillers` at the SPDR level / `.../billers/directory` at the Vanteo API level. |
| 11 | Add Electronic Biller | POST `/billpay/addElectronicBiller` | POST `/v1/billpay/billers` | VC-191 ("Link Invoice to Biller") | VC-199, VC-203 (BE: link an electronic biller) | Fully traced. Confirms draft name `linkBiller` (see §3) is the same feature as `addElectronicBiller`. |
| — | Create Bill Payment (ACH) | POST `/billpay/createPayment` | POST `/v1/billpay/payments` | VC-192 (Make a Payment), VC-193 (Review Payment Summary), VC-194 (Payment Success), VC-198 (Submit ACH Payment) | VC-204 (BE: submit ACH electronic payment) | Fully traced. Same SPDR/internal endpoint as #5 — one implementation serves both payment types. |

Other stories under VC-184 not tied to a single new endpoint: VC-197 ("Error States", spans all APIs above), VC-200 ("Schema: linked-electronic-biller table" — supports #9/#11).

---

## 3. Superseded / Draft Naming — Resolved

| Draft Name | Found In | Confirmed Current Name | Status |
|---|---|---|---|
| `POST /billpay/searchBillerDirectory` | `Vanteo_Connect_BillPay_ElectronicBillers_PRD_v1`, `Vanteo_BillPay_ElectronicBillers_UserStories_v1–v3` | `GET /billpay/searchBillers` (#10) | **Resolved** — Jira dev task VC-202 confirms `searchBillers` / `.../billers/directory` is the built endpoint. Update or archive the older PRD drafts to avoid confusion. |
| `POST /billpay/linkBiller` (unconfirmed name) | `Vanteo_Connect_BillPay_ElectronicBillers_PRD_v1` | `POST /billpay/addElectronicBiller` (#11) | **Resolved** — Jira dev task VC-203 and story VC-191 confirm `addElectronicBiller` is the built endpoint. |

---

## 4. Traceability Gaps Summary

| Gap | Impact |
|---|---|
| **Remove Biller has no Jira story or dev task** | Documented in PRD Part 1 but entirely absent from the current backlog. If it's in scope for the paper check epic, it needs to be written up and estimated. |
| **Cancel Payme
