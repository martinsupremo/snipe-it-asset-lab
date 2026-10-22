# Snipe-IT Asset Implementation — IT Asset Management for Office + Warehouse Inventory

At Marubeni in Miami I handled IT inventory on the warehouse floor: scanners, label
printers, and keeping the system in line with what was actually on the shelves. This
repo sets up Snipe-IT for that kind of office-plus-warehouse inventory, using
made-up assets and locations.

It covers asset intake, barcode labels, check-in/check-out, cycle counts, and
reconciling the physical count against the system, in one web app backed by
MariaDB.

**What this demonstrates:** Snipe-IT deployment (Docker + MariaDB), asset data modeling
(tags, categories, models, status labels, custom fields, locations), CSV bulk import,
barcode label workflow for Zebra-style printers and handheld scanners, check-in/check-out
SOPs, and scan-based audit/reconciliation procedures.

```
   ┌──────────────────┐        ┌──────────────────┐
   │  IT ADMIN (web)  │        │ WAREHOUSE STAFF  │
   │  intake, audits, │        │ handheld scanner │
   │  reports         │        │ (keyboard wedge) │
   └────────┬─────────┘        └────────┬─────────┘
            │  HTTPS / browser            │ scan asset tag
            └──────────────┬──────────────┘
                           ▼
                ┌─────────────────────┐        ┌──────────────────┐
                │   SNIPE-IT WEB APP  │───────▶│  LABEL PRINTER   │
                │   (Docker, :8080)   │ labels │  barcode / QR    │
                └──────────┬──────────┘        └──────────────────┘
                           │ SQL
                           ▼
                ┌─────────────────────┐
                │       MariaDB       │
                │ assets, users,      │
                │ locations, history  │
                └─────────────────────┘

   Asset lifecycle:  Purchase → Intake/tag → Ready to Deploy → Deployed
                     → (check-in) → Pending wipe/repair → Retired/Disposed
```

## Repo layout

| Path | Purpose |
|---|---|
| `docker-compose.yml` | Snipe-IT app + MariaDB stack (port 8080:80), volumes, env wiring |
| `.env.example` | Every env var the compose file expects (placeholders only) |
| `docs/setup-notes.md` | First-run setup: APP_KEY, migrations, admin user, exact categories/statuses/locations to create, barcode settings |
| `docs/architecture.md` | Design decisions: tag scheme, data model, custom fields, locations, barcode workflow, hardening |
| `docs/checkin-checkout-workflow.md` | SOP for checkout, checkin, overdue follow-up, lost/damaged handling |
| `docs/audit-reconciliation-procedure.md` | SOP for cycle counts, full audits, discrepancy resolution, sign-off |
| `templates/assets-import.csv.example` | Sample bulk-import CSV (8 fictional assets: laptops, peripherals, scanners, label printer) |

## Quick start

```bash
# 1. Configure environment (fill in real passwords + a real APP_KEY)
cp .env.example .env
docker run --rm snipe/snipe-it php artisan key:generate --show
#    -> paste the base64:... output into .env as APP_KEY

# 2. Start the stack
docker compose up -d

# 3. Create the first admin user (or finish setup in the web wizard)
docker compose exec app php artisan snipeit:create-admin \
  --first_name="Demo" --last_name="Admin" \
  --email="admin@example.local" --username="admin"
#    -> you will be prompted for a password; use a real one, never commit it

# 4. Open http://localhost:8080, log in, then create the categories,
#    status labels and locations listed in docs/setup-notes.md

# 5. Import the sample inventory
#    Admin > Import > upload templates/assets-import.csv.example
#    (rename to .csv first) and map the columns to Snipe-IT fields
```

## Security notes

- `.env` is **never committed** (see `.gitignore`). Only `.env.example` with
  placeholder values (`changeme-*`, a fake APP_KEY) lives in this repo.
- Database passwords are placeholders; set strong unique values in `.env` on any
  real host. MariaDB is not published to the host network — only the app reaches it.
- All sample data is fictional: serials are `SN-FAKE-####`, no real people,
  no real inventory. Real exports (which contain real serials) are gitignored.
- On a real deployment: put Snipe-IT behind HTTPS, restrict the admin panel by
  network/VPN, enable backups of the MariaDB volume and `storage/app`, and review
  user permissions — not everyone needs delete rights on assets.

## What's been checked

- `docker-compose.yml` is valid YAML, defines the `app` and `db` services with
  port `8080:80`, and every variable it uses is defined in `.env.example`.
- The CSV import template has a header row and 8 example rows, all with the
  same number of columns.
- Still to do on a Docker host: start the stack, generate an APP_KEY, run the
  first-time web setup, import the CSV, and print and scan a real barcode
  label. `docs/setup-notes.md` walks through each step.
