# CareerBridge contributor handoff

## Scope and architecture
- Source export of the existing CareerBridge Sites application; TypeScript/React/Vinext on Cloudflare Workers with D1 and Drizzle migrations.
- Read README.md and MARKETPLACE.md before changing behavior. Application and route handlers live under app; lib contains marketplace, payments, delivery, refunds and AI logic.
- Preserve server-side ownership checks, approved-only mentor listings, booking uniqueness, payment verification and idempotency. Money uses integer minor units where implemented.
- Production identity is trusted only behind the Sites gateway. Independent deployment requires a validated authentication replacement.

## Validation
- Run the README check commands after installing the locked dependencies. API tests mock providers and identity; live provider behavior requires separate tests.
- Keep credentials and production data out of source control. Do not modify Git history or deploy without user authorization.

## Export status (2026-10-27)
- Added project README, GitHub upload guide, deployment limitations, safe configuration example and ignore rules.
- Removed generated TypeScript cache and original Site project ID from the export; retained all application source and migrations.
- Automatic disbursement, unattended notification retries and independent hosting remain unfinished. Provider activation and signed-in browser verification remain separate work.
