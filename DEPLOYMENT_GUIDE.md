# Zain G Cafe deployment guide

This project is built for a Cloudflare-compatible Next.js/Vinext runtime and includes:

- D1 database migrations in `drizzle/`
- R2 product-image storage
- password-protected admin dashboard
- public database-backed menu

## Required production bindings

- D1 database binding: `DB`
- R2 bucket binding: `MEDIA`
- `ADMIN_EMAILS`: comma-separated administrator emails
- `ADMIN_INITIAL_PASSWORD`: temporary first-login password
- `ADMIN_SESSION_SECRET`: a random 64-character secret

## Installation

1. Install Node.js 22 or newer.
2. Run `npm install`.
3. Create the D1 database and R2 bucket in Cloudflare.
4. Configure the bindings above in your hosting environment.
5. Apply every SQL file inside `drizzle/` in filename order.
6. Run `npm run build`.
7. Deploy the generated Worker and static assets using your Cloudflare-compatible host.

The first administrator signs in at `/admin/login` using the configured email and temporary password. The dashboard immediately requires a stronger replacement password.

## Important

Traditional shared PHP hosting cannot run this application directly. Use Cloudflare Workers, a compatible serverless Node/Worker host, or ask your hosting provider to confirm support for Next.js server routes, D1-compatible SQL storage and object storage.
