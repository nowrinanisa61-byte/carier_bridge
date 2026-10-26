# CareerBridge

**Learn from experience. Build your career.**

A career mentorship platform connecting graduates with active industry experts and experienced senior professionals, with AI-assisted career preparation.

[Live application](https://careerbridge.almahfuz.chatgpt.site)

## Features

- Student and mentor profiles, mentor applications and administrator approval.
- Mentor discovery, availability, booking requests and meeting links.
- Booking conflict protection, cancellation and completed-session reviews.
- SSLCommerz checkout and verified payment callbacks.
- In-app notifications, optional Resend email and Twilio SMS verification/alerts.
- Eligible refund requests, earnings calculations and withdrawal tracking.
- AI career roadmaps, approved mentor matching, CV text review and mock interviews in English or Bangla.

## Technology

TypeScript, React, Vinext, Tailwind CSS, Cloudflare Workers, D1/SQLite and Drizzle. AI uses the OpenAI Responses API.

## Getting started

Install Node.js 22.13+ (Node 24 recommended for the SQLite-based tests) and the package-manager version declared in `package.json` (`pnpm@11.25.0`). From the project root:

```sh
pnpm install --frozen-lockfile
pnpm dev
```

Open the URL printed by the development server. The portable development profile provides a loopback-only simulated sign-in. This is not production authentication. See [platform details](docs/PLATFORM.md) and [deployment notes](docs/DEPLOYMENT.md) for runtime requirements.

## Checks

```sh
node --experimental-strip-types lib/marketplace.test.ts
node tests/marketplace-api.mjs
node tests/ai-api.mjs
pnpm exec tsc --noEmit
pnpm build
```

API tests use mock identity, in-memory SQLite and mocked external providers. They do not prove real payment, email/SMS delivery or AI connectivity.

## Integration status and limitations

Provider adapters are implemented; real operation requires provider credentials and end-to-end verification. Refunds require eligibility and explicit activation. Mentor bank/bKash transfers are manual; withdrawal tracking does not transfer funds. Current commission is 10%, with a seven-day earnings hold. There is no unattended notification retry worker.

Production authentication depends on the ChatGPT Sites gateway. **This source export is ready for GitHub, but is not a standalone deployment package.** Independent hosting requires a secure authentication replacement and Cloudflare configuration. GitHub Pages cannot run the server or database.

## Documentation

- [Upload to GitHub](docs/GITHUB_UPLOAD.md)
- [Deployment and configuration](docs/DEPLOYMENT.md)
- [Marketplace behavior and provider setup](MARKETPLACE.md)
- [Platform development reference](docs/PLATFORM.md)

No credentials, production database records, dependencies or build outputs are included. See `.env.example` for configuration names; it is a reference, not automatic runtime setup.
