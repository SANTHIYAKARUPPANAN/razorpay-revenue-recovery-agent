# Recovery Radar

Recovery Radar detects failed Razorpay payments, selects a deterministic recovery strategy, creates a replacement payment link, and gives payment teams a clear recovery worklist.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 5000)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `artifacts/recovery-radar/` — React + Vite dashboard
- `artifacts/api-server/src/routes/recovery.ts` — dashboard, simulation, and status routes
- `artifacts/api-server/src/routes/webhook.ts` — signed Razorpay webhook handler
- `artifacts/api-server/src/lib/recovery.ts` — rule-based decisions, Razorpay links, seed data, and recovery activity
- `lib/db/src/schema/recovery.ts` — PostgreSQL tables
- `lib/api-spec/openapi.yaml` — source-of-truth API contract

## Architecture decisions

- The frontend uses generated React Query hooks from the OpenAPI contract rather than hand-written request types.
- The app stores amounts in rupees for readable dashboard values and converts to paise only at the Razorpay API boundary.
- Demo simulation intentionally disables Razorpay customer notifications; the generated link is displayed for manual testing only.
- Webhook requests capture the raw JSON body so Razorpay signatures can be verified before any recovery work runs.

## Product

The dashboard surfaces at-risk and recovered revenue, a payment queue with rule reasoning and generated recovery links, a recent activity stream, demo failure simulation, and webhook readiness checks. Recovery status can be marked recovered or abandoned from the payment detail view.

## User preferences

- Keep the recovery workflow demo-safe: log/display messages instead of sending real email, SMS, or WhatsApp notifications.

## Gotchas

- The API server needs `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`, and `RAZORPAY_WEBHOOK_SECRET` in Replit Secrets for live simulation and webhook processing. Recovery decisions are rule-based and require no AI key.
- The frontend build command requires workflow-provided `PORT` and `BASE_PATH`; use the artifact workflow for normal runs.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
