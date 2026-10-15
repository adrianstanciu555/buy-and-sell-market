
# Project Context for AI Assistance

## Project Overview

- **Name:** buy-and-sell-market
- **Purpose:** A managed marketplace where sellers send items to the platform's warehouse. The platform verifies, stores, and fulfills inventory so sellers have limited direct interaction with buyers.
- **Domain:** Consumer-to-consumer marketplace with centralized warehousing and fulfillment.
- **Initial scale target:** 10,000 daily active users and 50,000 listed items. Daily active users do not define peak concurrent traffic; peak load and growth targets must be established before launch.
- **Project stage:** Requirements and production architecture are proposals. No implementation, cloud provider, service-level objective, or external provider has been confirmed.
- **Operating assumption:** The platform handles physical custody and transaction operations. Whether legal ownership transfers to the platform at any point, and the seller payout/return policies, require explicit business decisions.

## Technical Stack

- **Frontend:** Proposed React with TypeScript for buyer, seller, admin, and warehouse operations interfaces.
- **Backend:** Proposed Node.js with Express as a modular monolith exposing a REST API. Keep catalog, identity, inventory, order, payment, fulfillment, and administration as clear modules; split services only when measured scale or team ownership warrants it.
- **Database:** Proposed managed PostgreSQL for transactional data. Core records include users and roles, listings, inventory units and movements, warehouse locations, orders, payment references, shipments, returns, and audit events. Use indexes and transaction constraints to protect inventory and order consistency.
- **Infrastructure:** Proposed cloud deployment with separate development, staging, and production environments. Store item images in object storage and serve them through a CDN. Use a managed queue for retryable background work such as notifications and shipping-provider updates. Add caching only for measured hot paths; do not start with microservices or a dedicated search cluster without evidence they are needed.
- **External services:** Payment, shipping-label/tracking, email, and identity providers are undecided. Integrate through provider interfaces and use sandbox accounts in non-production environments. Do not store payment card details.
- **Testing tools:** Proposed Vitest for unit and API tests, Playwright for critical browser journeys, and a load-testing tool such as k6 for agreed traffic targets. These are proposals, not tools already configured in the repository.

## Key Features & Modules

1. **Identity and access:** Registration, sign-in, account recovery, and server-side role authorization for Buyers, Sellers, Admins, and Warehouse Operators.
2. **Seller catalog:** Sellers create listings and send associated items to the warehouse. A listing is not purchasable until required inventory has been received and checked in.
3. **Warehouse operations:** Receive inbound seller shipments, identify and count items, record condition and discrepancies, assign storage locations, and track every inventory movement. Operators pick, pack, and dispatch paid orders and process returns.
4. **Buyer catalog and checkout:** Buyers search and filter available items, view accurate stock/status, place orders, and receive order and shipment updates.
5. **Orders, payments, and seller settlement:** Record order state and payment-provider references, prevent overselling, handle cancellation/refund flows, and calculate seller proceeds. Settlement timing and fee rules remain to be defined.
6. **Fulfillment and returns:** Create shipment records and tracking updates; support return authorization, warehouse receipt, inspection, restocking or write-off, and refund status.
7. **Administration and reporting:** Admins manage users, moderate listings, monitor orders and inventory exceptions, and review audit history. Warehouse staff should use a narrower operational role rather than general Admin access.

## Current Testing Approach

- **Manual testing:** No established test process or test environments yet. Plan role-based exploratory testing for the catalog, warehouse receiving, checkout, dispatch, returns, and admin workflows.
- **Automation:** Not yet configured. Proposed coverage: unit tests for domain rules; API and database integration tests for permissions, inventory, order transitions, and retries; a small Playwright suite for critical end-to-end journeys.
- **Performance:** No measured baseline or SLO exists. Test with 50,000 catalog items and a representative data set. Define peak concurrent users, request mix, and growth headroom from expected usage, then load-test browse/search, item detail, checkout, and warehouse operations against those targets.
- **Reliability:** Test database backup/restore, queue retry and dead-letter handling, provider timeouts, duplicate/out-of-order webhooks, and recovery from partial fulfillment failures.
- **Security:** No implementation exists yet. Test authentication, least-privilege role access, object-level authorization, input validation, rate limiting on sensitive endpoints, secure session handling, and auditability of administrative and inventory changes. Keep secrets out of source control and use synthetic data outside production.

## Pain Points & Known Issues

- **Recurring bugs:** None known; the application has not been implemented.
- **Performance bottlenecks:** None measured. Potential risks to validate include catalog query performance, image delivery, checkout contention, and warehouse inventory reconciliation.
- **Testing gaps:** Peak concurrency, service objectives, return/refund policy, seller settlement rules, warehouse exception handling, and provider contracts need definition before production readiness can be assessed.
- **Operational risks:** Inventory counts can diverge from physical stock; duplicate or delayed provider events can corrupt state if operations are not idempotent; warehouse outages can delay dispatch. Define reconciliation, exception queues, and operational ownership.

## Business Rules & Constraints

- **Critical user journeys:** Seller creates a listing and ships items to the warehouse; warehouse receives and inspects items and makes accepted stock available; Buyer orders available stock and pays; warehouse picks, packs, and dispatches the order; Buyer tracks delivery or requests a return; warehouse inspects returns and the platform updates stock/refund state.
- **Inventory:** Track each physical unit or controlled quantity by listing/SKU, condition, and warehouse location. Only received, inspected, sellable stock is available for checkout. Inventory reservation and order placement must be atomic enough to prevent overselling. Every adjustment must have a reason and audit record.
- **Listing and custody:** Sellers may edit listing details subject to inventory and order state. Warehouse receipt records expected versus actual quantity and condition. Discrepancies require resolution before affected stock is sellable. Custody, ownership, loss/damage liability, and disposal rules require legal/business definition.
- **Orders and money:** Payment and refund states must be reconciled with the payment provider; sensitive card data remains with that provider. Define cancellation, returns, refunds, platform fees, and seller payout timing before implementing settlement.
- **Permissions:** Buyers access their own orders; Sellers access their own listings, inbound shipments, and settlement information; Warehouse Operators access assigned warehouse workflows; Admin actions are restricted and audited. Enforce authorization on the server for every protected resource.
- **Compliance requirements:** Applicable consumer protection, privacy, tax, and payment requirements depend on launch countries and are not yet determined. Complete a jurisdictional review before production launch.
- **Performance and availability objectives:** None approved. Establish measurable SLOs for availability, latency, data recovery, and peak load before launch; daily active user count alone is insufficient to size the system.
- **Security policies:** Use secure password hashing, TLS in transit, encryption at rest where supported, least-privilege access, managed secrets, audit logs, and a documented backup/restore process. Minimize retained personal data.

## Sample Data & User Roles

- **Buyer:** Browse available items, place and pay for orders, view their own order/shipment status, and request returns.
- **Seller:** Create listings, prepare inbound shipments, view receipt discrepancies, and see sales and settlement status for their own items.
- **Warehouse Operator (proposed additional role):** Receive, inspect, locate, pick, pack, dispatch, and process returned inventory. Access is limited to assigned operational tasks and locations.
- **Admin:** Manage accounts and listings, resolve exceptions, configure marketplace operations, and review audit records. Admin access is not a substitute for routine warehouse permissions.
- **Test accounts:** Use separate synthetic accounts for every role and include unauthorized cross-account attempts in tests. Never use production credentials in test environments.
- **Data sets:** Generate at least 50,000 synthetic listings plus inventory units, warehouse locations, orders, shipments, returns, and lifecycle edge cases. Include unavailable, damaged, reserved, sold, and returned stock.
- **Environments:** Proposed development, staging, and production environments. Staging should use synthetic data and payment/shipping sandbox integrations; production access and data must be restricted.
