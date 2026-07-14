# Figma Documentation

## Purpose

This folder serves as the Product Management index for all Figma designs related to the Vanteo Connect Bill Pay module.

Figma remains the source of truth for UI/UX design. This folder provides traceability between product requirements, design, engineering, and testing.

## Objectives

* Maintain an inventory of all Bill Pay screens.
* Link each screen to its corresponding PRD requirement.
* Link each screen to the implementing JIRA story.
* Track design status throughout the product lifecycle.
* Document significant design decisions.

## Folder Contents

| File                     | Purpose                                       |
| ------------------------ | --------------------------------------------- |
| `README.md`              | Overview of the Figma documentation structure |
| `screen-inventory.md`    | Master inventory of all Bill Pay screens      |
| `design-decisions.md`    | Record of key UX and UI decisions             |
| `component-inventory.md` | Reusable components and design patterns       |
| `release-mapping.md`     | Mapping of screens to product releases        |

## Screen Naming Convention

Assign a permanent identifier to every screen.

Example:

| Screen ID | Screen Name          |
| --------- | -------------------- |
| UI-001    | Bill Pay Home        |
| UI-002    | Search Billers       |
| UI-003    | Biller Details       |
| UI-004    | Payment Amount       |
| UI-005    | Review Payment       |
| UI-006    | Payment Confirmation |

Screen IDs should remain stable even if screen names change.

## Traceability

Each screen should map to the following artifacts:

* Product Brief
* PRD Requirement
* JIRA Story
* GoSPDR API (where applicable)

Example:

```text
Product Brief
      ↓
PRD Requirement (BP-PAY-001)
      ↓
JIRA Story (BILL-201)
      ↓
Figma Screen (UI-005)
      ↓
GoSPDR API
```

## Design Principles

* Design every customer journey, including error and exception states.
* Keep screen names consistent with product terminology.
* Reuse components whenever possible.
* Document significant design decisions.
* Ensure every screen supports a defined product requirement.

## Related Documentation

* `/product/product_brief.md`
* `/prds/`
* `/jira/traceability-summary.md`
* `/integrations/api-inventory.md`


