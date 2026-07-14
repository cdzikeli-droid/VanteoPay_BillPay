# Bill Pay Ecosystem Overview

## Purpose

This document provides a high-level overview of the systems that support the Vanteo Connect Bill Pay feature.

## Architecture

```
Vanteo Connect Mobile App
          │
          ▼
     SPDR Bill Pay APIs
          │
          ▼
  Galileo Bill Pay Platform
          │
          ▼
       Sponsor Bank (FBOL)
```

## Components

### Vanteo Connect
- Customer-facing mobile experience
- Payment initiation
- Payment status
- Notifications

### SPDR
- Bill Pay API provider
- Routes requests to Galileo
- Returns biller and payment data

### Galileo
- Bill Pay processor
- Biller directory
- Payment scheduling and execution
- Payment status

### FBOL
- Sponsor bank
- Compliance and operational oversight

## High-Level Flow

1. Customer signs in to Vanteo Connect.
2. Customer searches for a biller.
3. Customer submits a bill payment.
4. Vanteo sends the request to SPDR.
5. SPDR forwards the request to Galileo.
6. Galileo processes the payment.
7. Payment status is returned to Vanteo through SPDR.
8. Vanteo displays the payment status to the customer.

## Related Documents

- `/product/product_brief.md`
- `/prds/`
- `/requirements/business-rules.md`
- `/requirements/payment-state-model.md`
- `/integrations/api-inventory.md`
