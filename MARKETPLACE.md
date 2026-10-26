# CareerBridge marketplace

## Implemented
- Anonymous visitors can browse approved, user-submitted mentor profiles.
- ChatGPT sign-in supplies server-trusted identity.
- Members choose student or mentor participation in their career profile.
- Mentors submit experience, skills, fees and dated availability for owner review.
- Only the account matching the runtime `ADMIN_EMAIL` can approve applications.
- Students request a 60-minute slot; the database's unique slot key blocks competing bookings.
- Mentors accept with an HTTPS meeting link or decline. Students can cancel.
- Session participants see their bookings; other users cannot access or mutate them.
- Completion is restricted to the mentor after the scheduled end. Students review completed sessions once.
- Ratings are calculated from submitted reviews. There are no seed mentors or fabricated ratings.

## Runtime configuration
`ADMIN_EMAIL`: the platform owner's authenticated email, set in Sites runtime configuration. Missing configuration disables administrative actions; there is no automatic first-user administrator.
`DB`: provisioned D1 binding from the hosting manifest.

## Payment and delivery
Displayed fees are mentor quotes. SSLCommerz checkout is implemented and requires merchant activation as described below. Meetings use the mentor-supplied external HTTPS link. In-app notifications refresh automatically. Email and SMS adapters are implemented as described below; video hosting is external.

## Existing prototype records
The original workspace table remains intact for history. Its mock bookings are not migrated into real session records. The old write API returns 410; clients cannot approve themselves through it.

## Validation
Run `node --experimental-strip-types lib/marketplace.test.ts` for pure validation checks.
Run `node tests/marketplace-api.mjs` for actual API handlers with mock platform identity and an in-memory SQLite database. It applies all migrations and exercises anonymous browsing, approval, participant isolation, CSRF, role permissions, concurrent slot conflicts, cancellation, completion and unique reviews.
Run `node node_modules/typescript/bin/tsc --noEmit` and the Sites build helper before publishing.

## Next extensions
Automatic bank/mobile-wallet disbursement, credential verification, training cohorts, and AI assessment remain future extensions.

## SSLCommerz setup
Hosted checkout, callback and IPN validation are implemented against the SSLCommerz v4 APIs: https://developer.sslcommerz.com/doc/v4/ . Activation requires runtime secrets `SSLCOMMERZ_STORE_ID` and `SSLCOMMERZ_STORE_PASSWORD`. Configure `SSLCOMMERZ_MODE=sandbox` for sandbox merchant credentials; set `live` only after sandbox verification using live merchant credentials. `SITE_URL` must be the canonical HTTPS site origin.

Merchant IPN URL: `https://careerbridge.almahfuz.chatgpt.site/api/payments/ipn`.
Return URL: `https://careerbridge.almahfuz.chatgpt.site/api/payments/callback`.

Only the student may pay a future confirmed session. Amount comes from the stored booking. One gateway transaction per booking prevents duplicate checkout creation; an existing pending checkout can be resumed. The student enters billing details, which go to SSLCommerz and are not persisted in this database. Gateway URLs are restricted to HTTPS SSLCommerz hosts. Callbacks only update payment state after server-to-server validation of transaction ID, amount, currency and success status. Risk-flagged payments are held for manual review. Duplicate callbacks are idempotent, and a paid state is never downgraded by a delayed failure callback.

Failed/cancelled payment attempts are terminal for that booking: after cancellation, make a new booking to retry. An indeterminate checkout-initiation error remains locked to avoid a second charge and requires merchant-side reconciliation. Eligible pre-session paid cancellations create a full-refund reservation and trigger the refund API when enabled. Other paid cancellations require administrator assistance. Unattended reconciliation and external mentor transfers are not automated.

## In-app notifications
Notifications derive from current application, booking and payment states. They are a latest-update inbox, not a complete event audit log. Read receipts are persistent and scoped to the authenticated user. The browser polls every 30 seconds while visible. Email and SMS adapters require provider activation; push notifications are not implemented.

Payment API tests use a mocked gateway with the real request handlers and SQLite. They do not prove connectivity with an actual SSLCommerz merchant account. Browser sign-in testing encountered a navigation timeout; authenticated browser flows and real sandbox checkout still require end-to-end verification.

## Email and SMS notifications
Use `RESEND_API_KEY` and `RESEND_FROM_EMAIL` (verified sending domain) for email. Use `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_FROM_NUMBER` and `TWILIO_VERIFY_SERVICE_SID` for SMS and phone ownership verification. Sender registration, recipient-country permissions, account funding and provider rules must be configured with Twilio. Secrets belong in runtime configuration only.

Members explicitly opt in to email sent to their authenticated email address. SMS requires saving an E.164 number, completing Twilio Verify, then opting in. Verification initiation has an atomic one-minute cooldown. A reconciliation pass places each current booking, payment, refund and payout status in a durable deduplicated outbox before dispatch. Repeating the pass does not recreate the same user/channel/state job. Delivery dispatch claims jobs before sending; email requests use Resend idempotency keys. Unknown provider outcomes are marked for reconciliation and are not automatically resent. `Provider accepted` is not a delivery receipt. Queues are processed on app mutations and by the admin control; there is no unattended retry worker.

## Automatic refund requests
`AUTO_REFUNDS_ENABLED=true` allows gateway submission when payment credentials are configured. Full refunds apply only to paid, confirmed sessions cancelled before the start. A database batch validates eligibility with a conditional insert, reserves a unique refund and cancels the session atomically. The verified `bank_tran_id` is stored during payment validation. Refund initiation uses a unique `refund_trans_id`; only a verified refund-status response marks a refund completed. The user/admin Check gateway status action reconciles processing refunds. Unknown initiation outcomes require manual reconciliation to avoid a second refund.

SSLCommerz documents IP registration for live refunds. This must be resolved with the merchant/provider before enabling live refunds from the hosting environment; deployment alone does not prove the environment meets that requirement.

## Mentor earnings and withdrawals
Default policy: 10% commission and a seven-day hold after the 60-minute session ends. Only live Paid transactions with Completed sessions qualify; sandbox funds never become withdrawable. Amounts are stored as integer minor units. A conditional database insert and unique booking constraint prevent forged balances, early payout requests and duplicate withdrawals. Payout details are accessible only to their owner and the administrator.

The admin may approve a request and record a real externally completed bank/bKash transfer with a unique reference. `Recorded as paid` is a manual accounting record, not provider-confirmed settlement. No automatic disbursement connector or recipient KYC verification has been configured. That requires an approved payout provider and its actual API contract; this release does not fabricate one.

## AI career studio

Four signed-in tools: career roadmap, approved mentor matching, pasted-text CV review, and five-question text mock interviews with answer feedback and final debrief. English and Bangla output. No file upload or voice mode. All results are AI-generated advice, not hiring decisions or verified current market data.

Server-only `OPENAI_API_KEY` activates `/api/ai`. Obtain it using the OpenAI Developers plugin's secure API-key workflow, then store it as a Sites runtime secret. Optional `OPENAI_MODEL` defaults to `gpt-4.1-mini-2025-04-14`. The Responses API uses strict structured output, a 45-second timeout, 2,600 output-token cap and `store:false`. Provider access/billing and a real request still require activation verification; configured does not mean tested.

Users explicitly consent before submitting input to OpenAI. Saved profile context is opt-in and limited to the authenticated user's goal, skills and bio. CareerBridge does not persist CV text, answers or responses; the UI retains them in component memory until leaving the page. OpenAI's own API retention policies still apply. Only quota counters, dates and timestamps are stored in `ai_usage`.

Atomic D1 counters enforce 20 attempts/user/day, 10 seconds between requests, and 500 attempts/site/day, resetting at 00:00 UTC (06:00 Bangladesh). Provider failures count. No automatic retries. The global quota may consume a user attempt even when the site limit blocks the request. Budgets should also be configured in the provider account before activation.

Mentor candidates are drawn from up to 100 approved directory entries, ordered by ID, with bounded public context. Returned cards are server-filtered to those exact IDs; recommendations do not guarantee availability. AI narrative can still be inaccurate. No booking, money movement or other tool action is granted to the model.

Validation: `node tests/ai-api.mjs` uses actual route handlers with in-memory SQLite and mocked OpenAI Responses. It covers auth/origin/consent/input limits, ownership, approved-only candidates, interview turns, malformed/incomplete provider responses, unavailable configuration and atomic quotas. It does not establish real model quality or external connectivity.
