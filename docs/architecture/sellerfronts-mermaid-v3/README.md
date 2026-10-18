# SellerFronts Mermaid Architecture Pack v3

Canonical, editable source for the SellerFronts **target architecture**. Treat the eleven PNG variations as historical references, not independent sources of truth. Regenerate visual previews from these five Mermaid source files.

These diagrams describe the intended target and migration direction; they do **not** prove that every component or control is already implemented in the repositories. Use the **Current code gaps** section below when planning work or assessing implementation status.

## Fixed target decisions

- One customer payment provider: a single platform Razorpay account for checkout, with Razorpay Route transfers to activated seller linked accounts. Sellers configure payouts, not per-seller checkout gateways. Juspay is a retire-or-fallback decision behind a provider-neutral PaymentGateway interface, not a customer checkout radio option.
- Tiered radius: free organic discovery within 5 km; buyers may widen the search tier (5–20 km, 20–100 km, city/region, All India). Paid visibility may widen eligible reach; seller delivery/serviceability remains a separate, hard checkout gate.
- Four launch revenue streams: paid visibility, category banners, custom-domain hosting, and BBB attendee-hours. Analytics, premium listings and support remain Phase 5/future.
- Shared tenancy decision: define a shared permission contract before Phase 1 for Seller Admin (own channel only) and Platform Operator (explicit cross-tenant permission). Keep reusable permission types/policy helpers separate from domain plugin services; each plugin enforces its own channel-aware access.
- Stock lifecycle: reserve at checkout, allocate after confirmed capture, release on cancel/timeout, and handle StockShortfallEvent.
- Payment reliability: verify webhook signatures against the raw body, persist an inbox event before business processing, deduplicate and process idempotently, reconcile late/missing events against the Razorpay API, and reverse applicable Route transfers during refunds.
- Operational recovery: worker retry/DLQ, webhook replay, tracing/logs and reconciliation views are first-class.

## Current code gaps

- Current config has dummyPaymentHandler; Juspay is wired, while the Multivendor Razorpay adapter is a seller-transfer integration, not the completed customer checkout gateway.
- Multivendor config contains a 10% platform fee and FEE SKU; 0% needs a code change and regression test.
- SellerGeo config is 20 km; target default organic tier is 5 km.
- Elasticsearch registration was commented out in the audited config; verify a compatible plugin is active before claiming search works.
- BBB API helper currently hard-codes SHA-256; make checksum configurable against server-supported algorithms. Persist-first webhook inbox, join rate limiting and attendee-hour reconciliation must be verified/implemented.
- Frontend promotion flag is off by default; enable after campaign, sponsored-label, wider-tier and exhausted-budget tests pass.

## Operating constraints

- All data mutations through Vendure Shop/Admin GraphQL API calls; never edit PostgreSQL records manually.
- Database schema changes through reviewed Vendure CLI migrations using the documented npx vendure migrate workflow; keep synchronize false outside disposable local databases.
- Gate Route rollout on a two-seller checkout test: no FEE line; captured amount reconciles to explicit shipping/tax allocations and seller transfer totals; idempotency, refunds and reversals are proven.
- BigBlueButton getMeetingInfo supports live/stuck-state reconciliation, but participant counts alone do not calculate attendee-hours; use per-attendee analytics callbacks for exact billing totals.
- In Mermaid sequence diagrams, keep each message on its own line and use plain, unnumbered message labels. Prefer `autonumber` over numbering text inside message labels, and avoid semicolons in labels to reduce parser ambiguity. Diagram 4 follows this style; it has been syntax-hardened, but should still be rendered in the Mermaid version used by the project before publishing generated previews.

## Diagrams

1. [Target platform architecture](01-sellerfronts-target-architecture.mmd)
2. [Customer journey](02-customer-journey.mmd)
3. [Multi-seller checkout and revenue](03-multiseller-checkout-and-revenue.mmd)
4. [BigBlueButton meeting, webhook and billing](04-bigbluebutton-session-and-billing.mmd)
5. [Plugin roadmap and gates](05-plugin-roadmap.mmd)

## References

- [Backend repository](https://github.com/ashishkpaul/buylits)
- [Frontend repository](https://github.com/ashishkpaul/buylits-frontend)
- [Vendure documentation index](https://docs.vendure.io/llms.txt)
- [Vendure migrations](https://docs.vendure.io/current/core/developer-guide/migrations)
- [Vendure multi-vendor marketplace guide](https://docs.vendure.io/current/core/how-to/multi-vendor-marketplaces.md)
- [Razorpay documentation index](https://razorpay.com/docs/llms-txt/)
- [Razorpay Route](https://razorpay.com/docs/build/llm-docs/payments/route/integration-guide.md)
- [Razorpay webhooks](https://razorpay.com/docs/build/llm-docs/webhooks.md)
- [BigBlueButton API](https://docs.bigbluebutton.org/development/api/)
- [BigBlueButton PHP API](https://github.com/bigbluebutton/bigbluebutton-api-php)
- [BigBlueButton plugins](https://github.com/bigbluebutton/plugins/blob/main/README.md)
