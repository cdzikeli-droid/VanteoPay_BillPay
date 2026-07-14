# Bill Pay UI Screen Traceability — Screen ID → PRD Requirement → Jira Story

**Product:** Vanteo Connect — Bill Pay
**Purpose:** Screen-level cross-reference for the Jira folder tracking Bill Pay UI screens. Maps each screen to its PRD functional requirement IDs and its Jira story name/ID.
**Last compiled:** 2026-07-14
**Source:** PRD functional requirement tables (`Vanteo_Connect_BillPay_PRD_v6.docx` for paper check, `Vanteo_Connect_Bill_Pay_PRD_Part2_ElectronicBiller.docx` for ACH) + `JIRA_VC_Full_Epic_Report 7.14.2026.xlsx`.

> **Requirement ID caveat:** FR numbering is **not globally unique** — both the paper-check PRD and the ACH PRD independently restart at FR-01. Always read an FR ID together with its PRD ID (e.g. "P1-ALT FR-01" ≠ "P2 FR-01"). Also, `Vanteo_Connect_Bill_Pay_PRD_Part1_PaperCheck.docx` (the fuller Part 1 spec) uses narrative section numbers instead of FR IDs — those are cited as "§x.x" below where that's the only source.

**PRD legend:** P1-ALT = `Vanteo_Connect_BillPay_PRD_v6.docx` · P1 = `Vanteo_Connect_Bill_Pay_PRD_Part1_PaperCheck.docx` · P2 = `Vanteo_Connect_Bill_Pay_PRD_Part2_ElectronicBiller.docx` · PB-01 = `GitHub BillPay Overview.docx` (Product Brief)

---

## Epic VC-67 — Paper Check Screens

| Screen ID | Screen Name | PRD ID | PRD Requirement ID | Jira Story Name | Jira Story ID |
|---|---|---|---|---|---|
| SCR-PCHK-01 | Bill Pay Landing Page | P1-ALT | FR-01–FR-08 | PCHK\| Bill Pay Home Landing Page | VC-132 |
| SCR-PCHK-02 | List of Payees | P1-ALT | FR-09–FR-17 | PCHK\| List of Payees | VC-133 |
| SCR-PCHK-03 | Payee Details | P1-ALT | FR-18–FR-23 | PCHK\| Select View Payee Details | VC-137 |
| SCR-PCHK-04 | Add a New Payee | P1-ALT | FR-24–FR-31 | PCHK\| Add a new Payee | VC-136 |
| SCR-PCHK-05 | Review Payee Details | P1-ALT | FR-32–FR-36 | PCHK\| Review Payee Details | VC-141 |
| SCR-PCHK-06 | Edit a Payee | P1-ALT | FR-37–FR-45 | PCHK\| Edit a Payee | VC-142 |
| SCR-PCHK-07 | Create a Payment | P1-ALT | FR-46–FR-53 | PCHK\| Create a Payment | VC-138 |
| SCR-PCHK-08 | Review & Confirm Payment | P1-ALT | FR-54–FR-59 | PCHK\| Review and Confirm Payment | VC-134 |
| SCR-PCHK-09 | Payment Success | P1-ALT | FR-60–FR-62 | PCHK\| Payment Success | VC-140 |
| SCR-PCHK-10 | Navigation & Error Handling *(cross-cutting, not a single screen)* | P1 | §3.8 (no FR ID — narrative only) | PCHK\| Navigation and Error Handling | VC-143 |
| SCR-PCHK-11 | View Payment History | P1 | §3.7 (no FR ID — narrative only) | BLPY\| Display Bill Pay Payment History | VC-135 *(filed under Epic VC-184, not VC-67 — flag)* |
| SCR-PCHK-12 | Remove a Saved Payee | P1 | §3.5 (no FR ID — narrative only) | **No Jira story found** | **—** |
| SCR-PCHK-13 | Cancel a Payment | P1 | §2.6 (no FR ID — narrative only) | BLPY\| Cancel Payment Flow | VC-195 *(filed under Epic VC-184, not VC-67 — flag)* |

---

## Epic VC-184 — ACH / Electronic Biller Screens

| Screen ID | Screen Name | PRD ID | PRD Requirement ID | Jira Story Name | Jira Story ID |
|---|---|---|---|---|---|
| SCR-BLPY-01 | Bill Pay Home — Biller List Card | P2 | FR-01 *(part of List of Billers group)* | BLPY\| Bill Pay Home Page w Biller List Card | VC-187 |
| SCR-BLPY-02 | List of Billers | P2 | FR-01–FR-09 | BLPY\| View List of Linked Billers | VC-196 *(overlaps with VC-187 — likely one capability split into two stories; confirm)* |
| SCR-BLPY-03 | Select a Biller | P2 | FR-10–FR-14 | BLPY\| Search and Select a Biller | VC-189 |
| SCR-BLPY-04 | Create Biller Customer Link | P2 | FR-15–FR-22 | BLPY\| Link Invoice to Biller | VC-191 |
| SCR-BLPY-05 | Pay a Biller | P2 | FR-23–FR-29 | BLPY\| Make a Payment | VC-192 |
| SCR-BLPY-06 | Review & Confirm Payment (reused/updated) | P2 | FR-30–FR-33 | BLPY\| REview Payment Summary | VC-193 |
| SCR-BLPY-06a | Submit ACH Payment *(same screen as SCR-BLPY-06, separate story for the submit action)* | P2 | FR-30–FR-33 | BLPY\| Submit ACH Payment | VC-198 |
| SCR-BLPY-07 | Payment Success (reused/updated) | P2 | FR-34–FR-36 | BLPY\| Payment Success | VC-194 |
| SCR-BLPY-08 | Error States *(cross-cutting, not a single screen)* | P2 | §10 (no FR ID — narrative only) | BLPY\| Error States | VC-197 |
| SCR-BLPY-09 | Payment Due Cards (Upcoming) | PB-01 | Not detailed in any PRD — only a V1 Scope bullet | BLPY\| Bill Pay Home Page with Payment Due Cards | VC-188 *(backed only by spike VC-206, not a committed dev task)* |
| SCR-BLPY-10 | View Biller Details | **Not in P2** | **No screen or FR group for this in the PRD** — Part 2's own Navigation Rules say biller taps skip directly to Pay a Biller, bypassing a details screen | BLPY\| View Biller Details | VC-190 *(flag: story with no PRD home)* |

---

## Flags carried over / new

1. **FR numbering collides across PRDs** (P1-ALT and P2 both restart at FR-01) — always pair an FR ID with its PRD ID.
2. **VC-190 (View Biller Details)** has no matching screen or requirement in Part 2 — the PRD explicitly describes skipping a details screen. Confirm whether this story should be cut or the PRD updated.
3. **VC-195 (Cancel Payment Flow)** and **VC-135 (Payment History)** are filed under the ACH epic (VC-184) even though their only PRD grounding is Part 1 (paper check). Part 2 additionally lists ACH cancellation as explicitly out of scope.
4. **SCR-PCHK-12 (Remove a Saved Payee)** has no Jira story or dev task at all.
5. **VC-187 and VC-196** look like a split of one "List of Billers" capability into two stories — worth confirming with the team before this becomes two parallel implementations.
6. **SCR-BLPY-09 (Payment Due Cards)** is only a line item in the original Product Brief, never fully specified in a PRD, and its only Jira backing is a spike, not a real dev task.

*Companion files in this folder: `GOSPDR_BillPay_API_Inventory.md` (API-level detail) and `BillPay_Traceability_Summary.md` (capability-level epic/PRD/Jira/API rollup).*
