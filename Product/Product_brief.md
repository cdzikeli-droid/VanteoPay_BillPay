# Bill Pay — Product Brief

## Elevator pitch
Enable Vanteo Connect customers to discover billers and pay U.S. bills directly from their Vanteo checking account in the mobile app, increasing primary account activity and customer retention.

## Overview
Bill Pay integrates SPDR's Bill Pay APIs and Galileo's Bill Pay platform to provide in-app discovery of billers, one-time and scheduled payments, payment tracking, and payee management. V1 focuses on core payment flows and transparent status reporting.

## Problem
Customers must currently leave the Vanteo app and use external bank bill-pay or biller websites for recurring and one-off bill payments, which reduces engagement and creates friction.

## Goal
Deliver a secure, intuitive Bill Pay V1 that:
- Lets customers discover and add billers/payees.
- Supports one-time payments and scheduled/recurring payments.
- Shows payment status and history.
- Improves engagement and increases activity on primary Vanteo accounts.

## Business objectives (examples — add numeric targets)
- Increase primary account usage by X% within 6 months.
- Improve customer retention rate by Y% among active users.
- Achieve monthly active Bill Pay users = Z within 3 months of launch.
- Maintain payment success rate >= 98%.

## Target users / personas
- Primary checking customers who prefer a single app for banking and bills.
- New customers consolidating accounts into Vanteo.
- Users who schedule recurring monthly bills (rent, utilities, credit cards).

## V1 Scope (In scope)
- Search and discover billers (by name, category, or biller code).
- Add payees (with identity/verification flow where required).
- Make one-time bill payments (immediate and scheduled for a future date).
- Schedule recurring payments (basic cadence: monthly, weekly - limited options in V1).
- View payment history and individual payment status (pending/processing/completed/failed).
- View list of added billers/payees.
- View upcoming payments due from linked billers.
- Basic notifications for payment initiation, success, and failure.
- Basic support/error messages for failed payments and retries.

## Out of scope (for V1)
- Advanced recurring rules (custom schedules beyond simple cadences).
- Bill presentment with extracted invoice items (detailed bill content).
- Multi-currency or international bill pay.
- Deep workflow for returned checks or complex disputes (handled by operations).
- Advanced fraud analytics beyond standard Galileo/SPDR offerings.

## User stories (examples)
- As a customer, I can search for my utility or credit card company and add it as a payee so I can pay them from my Vanteo account.
- As a customer, I can make a one-time payment to a biller and receive a confirmation in the app.
- As a customer, I can schedule a recurring monthly payment for a payee.
- As a customer, I can view the status of a payment (pending, completed, failed) and next steps if it failed.

## Acceptance criteria (examples)
- Search returns the expected biller when a user types the biller name or code within 2 seconds.
- A newly added payee is verified or set to pending verification with clear UI messaging.
- One-time payments initiated in-app reach the SPDR/Galileo flow and return a success code, and the payment appears in Payment History within 1 minute.
- Failed payments display a clear reason and suggested next steps, and an analytics event is logged.
- Notifications are sent for payment initiation and final status.

## Success metrics / KPIs
- Bill Pay adoption rate: % of active users who use Bill Pay within 30 days of release.
- First payment completion rate: % of users who complete a first bill payment after starting setup.
- Monthly active Bill Pay users (MAU).
- Payment success rate (settled vs. attempted).
- Number of customer support contacts related to Bill Pay (target baseline and target reduction).
- Net promoter score (NPS) or CSAT for Bill Pay flows.

## Dependencies & owners
- Vanteo Mobile App (iOS/Android): mobile engineering — owner: @mobile-team
- SPDR Bill Pay APIs: integration & contract — owner: @payments-infra
- Galileo Bill Pay platform: settlement & ledger — owner: @core-ledger
- FBOL sponsor bank approval and compliance: legal & partnerships — owner: @compliance
- Authentication & identity: OAuth/SSO and KYC flows — owner: @auth-team
- Notifications and transaction services: push & email — owner: @notifications
- Design assets & copy: Design/Content — owner: @design

Include API credentials, sandbox endpoints, and contract/SLAs in the internal integration doc (link to integration spec).

## Integration notes (technical)
- Use SPDR sandbox for functional testing; escalate to production only after FBOL bank sign-off and compliance checks.
- Required webhooks:
  - Payment status update webhook -> Mobile backend -> push notifications + update payment status in DB.
- Idempotency: ensure all payment calls use unique idempotency keys to avoid double payments.
- Error mapping: map SPDR/Galileo error codes to user-facing messages and internal error types for analytics.
- Data retained: store only necessary payee metadata and tokenized references, do not persist sensitive payment credentials.

## Non-functional requirements
- Security: all API calls must use TLS 1.2+; tokenization for payee/payment identifiers; follow PCI scope guidance — do not store PAN.
- Performance: search responses < 2s; payment initiation < 3s to acknowledge.
- Reliability: retries with exponential backoff for transient gateway failures; monitoring for webhook delivery failures.
- Privacy: comply with US data privacy expectations and bank-specific requirements.

## Monitoring & analytics (events to track)
- billpay_search_performed (query, results_count, latency)
- billpay_payee_added (user_id, payee_id, verification_status)
- billpay_payment_initiated (user_id, payment_id, amount, scheduled_date)
- billpay_payment_status_changed (payment_id, from_status, to_status, provider_code)
- billpay_payment_failed (payment_id, failure_code, failure_reason)
- billpay_notification_sent (type, success)
Report dashboards: success rate, latency, errors, volumes by biller.

## Support & operational playbook
- Include runbook for common failure scenarios: payment rejected, missing payee, webhook not received.
- Define SLA for support triage and escalation to payments/integration teams.
- Provide templated user messages for failed payments and bank return reasons.

## Risks & mitigations
- Sponsor bank (FBOL) approvals delay -> mitigation: secure approvals early; allow sandbox demo without production approval.
- Payment settlement delays or reversals -> mitigation: surface clear UI states and notify customers; include holds in transaction history.
- Fraud / abuse risk -> mitigation: allow velocity checks, link to fraud team, hold high-risk payments.
- Integration complexity with SPDR/Galileo -> mitigation: allocate dedicated integration sprints and contract test matrices.

## Launch & rollout plan (suggested)
- Internal alpha: feature flag in mobile app + QA + payments sandbox (1 sprint).
- Pilot: limited percentage of customers (5–10%) with monitoring and support on-call (2–4 weeks).
- Gradual rollout: 25% -> 50% -> 100% pending KPIs and incident rates.
- Post-launch: 30/60/90-day review on KPIs, support tickets, and UX improvements.

## Timeline (example)
- Week 0–2: Integration & design alignment; secure FBOL sandbox access.
- Week 3–6: Implement search, add payee, one-time payments.
- Week 7–8: Recurring payments & notifications.
- Week 9: End-to-end QA, compliance checks, and pilot prep.
- Week 10–12: Pilot + monitoring + iterate.

## Open questions / decisions
- What exact recurring cadence options do we support in V1? (monthly only vs multiple frequencies)
- What are the legal/data-retention constraints from the sponsor bank?
- Who signs off when a payment provider test fails — payments infra or compliance?

## Appendix / links
- Integration spec (link to internal doc)
- Design mockups (link)
- SPDR sandbox credentials (location)
- Galileo settlement doc (location)
