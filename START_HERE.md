# Zain G Cafe — Local Setup, Database and Hosting Guide

This ZIP contains the complete website, customer ordering experience, admin dashboard, database schema, migrations and image-upload backend.

## 1. What this project uses

- React 19 and Next.js-compatible Vinext
- TypeScript and Vite
- Cloudflare Workers runtime
- Cloudflare D1 SQL database, binding name `DB`
- Cloudflare R2 image storage, binding name `MEDIA`
- Drizzle ORM and numbered SQL migrations in `drizzle/`
- Node.js 22.13 or newer

The recommended independent host is **Cloudflare Workers**. Ordinary PHP/cPanel shared hosting cannot run this project without rewriting its server, database and upload layers.

## 2. Important files

| File or folder | Purpose |
| --- | --- |
| `app/` | Website pages, admin dashboard and API routes |
| `db/schema.ts` | Database table definitions |
| `drizzle/` | Database migrations and starter menu data |
| `worker/index.ts` | Cloudflare Worker entry point |
| `public/` | Static assets |
| `.env.example` | Local secret-variable template |
| `wrangler.local.jsonc` | Local D1/R2 configuration for migration commands |
| `wrangler.production.example.jsonc` | Production Cloudflare configuration template |
| `scripts/apply-migrations.mjs` | Applies SQL files in filename order |

## 3. Run on localhost — Windows

### Step 1: Install software

Install:

1. Node.js 22 LTS or newer from <https://nodejs.org/>
2. A code editor such as Visual Studio Code

Restart the computer after installing Node.js if `node` is not recognized.

### Step 2: Extract and open the project

1. Extract the ZIP.
2. Open the extracted `Zain-G-Cafe-Complete` folder.
3. Right-click inside the folder and choose **Open in Terminal**, or open it in Visual Studio Code and select **Terminal → New Terminal**.

Check Node.js:

```powershell
node --version
npm --version
```

Node should show version 22.13 or newer.

### Step 3: Install dependencies

```powershell
npm ci
```

This can take several minutes on the first run. Do not upload `node_modules` to hosting.

### Step 4: Create local environment variables

Copy `.env.example` and rename the copy to `.env.local`.

Generate a private session secret:

```powershell
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

Paste the generated value into `ADMIN_SESSION_SECRET` in `.env.local`. Set your owner email and a temporary first-login password. Never publish or send `.env.local` to anyone.

### Step 5: Create the local database tables and starter data

Run this **once** for a new local database:

```powershell
node scripts/apply-migrations.mjs site-creator-d1 --config wrangler.local.jsonc
```

This creates products, categories, administrator accounts, orders, reviews, inquiries, branches, delivery areas and payment records. It also inserts the removable sample menu.

If you intentionally want a completely fresh local database later, stop the local server, remove the local `.wrangler` folder, and run the migration command again. This deletes local-only data, so do not do it for production.

### Step 6: Start the website

Use this Windows-compatible command:

```powershell
npx vite --host 0.0.0.0
```

Open:

- Website: <http://localhost:5173>
- Menu: <http://localhost:5173/menu>
- Admin login: <http://localhost:5173/admin/login>

Press `Ctrl + C` in the terminal to stop the server.

## 4. Local administrator accounts

The sample migration includes:

| Email | Role | Temporary password |
| --- | --- | --- |
| `ansii0964@gmail.com` | Owner | Value of `ADMIN_INITIAL_PASSWORD` in `.env.local` |
| `zainali@gmail.com` | Owner | `admin123` |
| `ali123@gmail.com` | Manager | `admin123` |

Every temporary account should change its password immediately. The owner can edit, disable or delete accounts from **Admin Users**. Managers cannot access that section.

## 5. Recommended production hosting — Cloudflare

### Step 1: Create and verify a Cloudflare account

Create an account at <https://dash.cloudflare.com/>. A paid R2 or Images plan may be required depending on your usage. Keep billing and ownership under the business owner's account.

### Step 2: Sign in from the project terminal

```powershell
npx wrangler login
npx wrangler whoami
```

The first command opens Cloudflare in your browser.

### Step 3: Create the production D1 database

```powershell
npx wrangler d1 create zain-g-cafe-db
```

Cloudflare returns a `database_id`. Copy that exact ID.

### Step 4: Create the production R2 image bucket

```powershell
npx wrangler r2 bucket create zain-g-cafe-media
```

R2 stores product, deal and gallery images uploaded through the dashboard.

### Step 5: Create the production configuration

1. Copy `wrangler.production.example.jsonc`.
2. Rename the copy to `wrangler.production.jsonc`.
3. Replace `PASTE_YOUR_D1_DATABASE_ID_HERE` with the database ID from Step 3.
4. If you used different resource names, update `database_name` and `bucket_name` too.

Do not put passwords or the session secret directly in this configuration file.

### Step 6: Add production secrets

Run each command and enter the requested value privately:

```powershell
npx wrangler secret put ADMIN_EMAILS --config wrangler.production.jsonc
npx wrangler secret put ADMIN_INITIAL_PASSWORD --config wrangler.production.jsonc
npx wrangler secret put ADMIN_SESSION_SECRET --config wrangler.production.jsonc
```

Recommended values:

- `ADMIN_EMAILS`: `ansii0964@gmail.com`
- `ADMIN_INITIAL_PASSWORD`: a unique temporary password, not your permanent password
- `ADMIN_SESSION_SECRET`: generate it with the Node command from the localhost instructions

### Step 7: Apply database migrations

Run this **once** for the newly created production database:

```powershell
node scripts/apply-migrations.mjs zain-g-cafe-db --remote --config wrangler.production.jsonc
```

Do not repeatedly apply old migrations. For future website versions, apply only newly added migration files.

### Step 8: Build the production site

On Windows:

```powershell
npx vite build
```

On Linux with Bash, `npm run build` also runs the included artifact validation.

The production output is created inside `dist/`.

### Step 9: Deploy

```powershell
npx wrangler deploy --config wrangler.production.jsonc
```

Wrangler returns a public `workers.dev` address. Open it and test the website, menu, admin login, database status and image upload.

### Step 10: Add a custom domain

In Cloudflare:

1. Open **Workers & Pages**.
2. Select the `zain-g-cafe` Worker.
3. Open **Settings → Domains & Routes**.
4. Choose **Add → Custom Domain**.
5. Enter your domain or subdomain, such as `www.zaingcafe.pk`.
6. Follow the DNS confirmation shown by Cloudflare.

Enable HTTPS and redirect the non-preferred domain to the preferred one.

## 6. Production launch checklist

Before accepting real orders:

1. Log in to every seeded administrator account and replace the temporary password.
2. Disable or delete any administrator who should not have access.
3. Replace all dummy prices, descriptions and images.
4. Add the real address, opening hours, phone and WhatsApp number.
5. Verify every Google Maps link.
6. Set delivery charges, minimum order and expected delivery time.
7. Add only verified payment receiving accounts.
8. Connect debit/credit card payments only through a secure payment gateway. Never collect card numbers or security codes in this website or dashboard.
9. Place a complete test order and confirm it appears in **Admin → Orders**.
10. Test order status changes, review approval, inquiry replies and image uploads.
11. Test on Android, iPhone and desktop.
12. Create a database backup.

## 7. Database backup

Export the production D1 database:

```powershell
npx wrangler d1 export zain-g-cafe-db --remote --output zain-g-cafe-backup.sql --config wrangler.production.jsonc
```

Keep backups outside the public website folder. Create a backup before applying a new migration or making a major menu import.

## 8. Updating the hosted website later

For menu content, use the admin dashboard—no deployment is required.

For code/design changes:

```powershell
npm ci
npx vite build
npx wrangler deploy --config wrangler.production.jsonc
```

If a new numbered file was added to `drizzle/`, back up the database and apply only that new migration before deploying the code that needs it.

## 9. Common problems

### `node` or `npm` is not recognized

Install Node.js 22+, restart the terminal and run `node --version` again.

### Admin says database is unavailable

Confirm that the D1 binding is named exactly `DB`, then confirm all migrations were applied.

### Images do not save

Confirm that the R2 binding is named exactly `MEDIA` and that the `zain-g-cafe-media` bucket exists.

### Login returns to the login screen

Confirm `ADMIN_SESSION_SECRET` is present, use the correct email, clear site cookies and sign in again.

### Shared hosting/cPanel upload does not work

This is a full-stack Worker application, not a static HTML or PHP website. Use Cloudflare Workers or hire a developer to adapt the database, storage and server runtime for your host.

### Migration says a column or table already exists

That migration has already been applied. Stop and do not rerun the entire migration set against the same database.

## 10. Security rules

- Never upload `.env.local`, `wrangler.production.jsonc` with secrets, database backups or Cloudflare tokens to a public repository.
- Use a different password for each person.
- Keep at least one active owner account.
- Remove access immediately when a staff member leaves.
- Never put raw card data in D1, R2, inquiries, order notes or chat.
- Back up D1 before structural changes.

