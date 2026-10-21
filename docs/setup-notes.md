# First-run setup notes

Follow these in order on a Docker host. Estimated hands-on time: ~45–60 minutes.

## 1. Prerequisites

- Docker Engine 24+ with the Compose plugin (`docker compose version` works).
- No real credentials invented ahead of time — generate them during setup.

## 2. Environment file

```bash
cp .env.example .env
```

Fill in `.env` (the placeholders are **not** valid values):

- `APP_KEY` — generate exactly once, keep it secret:
  ```bash
  docker run --rm snipe/snipe-it php artisan key:generate --show
  ```
  Paste the full `base64:...` output as `APP_KEY`. If you lose it after data
  goes in, encrypted fields (two-factor secrets, custom encrypted fields)
  become unreadable — store it with your other secrets.
- `DB_PASSWORD` / `MYSQL_ROOT_PASSWORD` — strong unique passwords
  (e.g. `openssl rand -base64 24`).
- `APP_URL` — the URL you'll reach the app at (default
  `http://localhost:8080` for local use; a real hostname behind HTTPS in
  production).
- Mail settings — optional. Leave blank here; Snipe-IT works without
  outbound email (you lose check-in reminders and audit notifications).

## 3. Start the stack and run migrations

```bash
docker compose up -d
docker compose logs -f app   # wait until the app stops restarting
```

The `snipe/snipe-it` image runs migrations on first boot. Verify tables exist:

```bash
docker compose exec db mariadb -u root -p"$MYSQL_ROOT_PASSWORD" \
  -e "USE snipeit; SHOW TABLES LIKE 'assets';"
```

## 4. Create the admin user

Web wizard (recommended the first time — it also sets the default currency,
locale, and your company name):

1. Open `http://localhost:8080`.
2. Follow the setup wizard: site name, locale, currency, admin account.

Or via CLI if you prefer:

```bash
docker compose exec app php artisan snipeit:create-admin \
  --first_name="Demo" --last_name="Admin" \
  --email="admin@example.local" --username="admin"
# prompted for a password — use a real one; it is not stored in the repo
```

## 5. Seed the data model

Create these in Snipe-IT (names must be exact — the SOPs and sample CSV use them):

**Status labels** (Settings → Status Labels):

| Name | Type | Color | Deployable |
|---|---|---|---|
| Ready to Deploy | deployable | green | yes |
| Deployed | deployable | blue | yes |
| Pending | undeployable | orange | no |
| Out of Repair | undeployable | yellow | no |
| Broken - Not Fixable | archived | dark gray | no |
| Lost/Stolen | archived | red | no |
| Retired | archived | gray | no |

**Categories** (Assets → Categories):

- Laptops, Desktops, Peripherals, Barcode Scanners, Label Printers,
  Networking, Mobile Devices

**Locations** (Assets → Locations — parent → child):

- Office - HQ
  - Office - IT Closet
  - Office - Dock Office
- Warehouse
  - Warehouse - Receiving
  - Warehouse - Aisle 1
  - Warehouse - Shipping
  - Warehouse - Repair Bench

**Custom fields** (Settings → Custom Fields): IMEI, MAC address (Wi-Fi),
MAC address (Ethernet), Purchase order #, Warranty expiration,
Data wipe verified (Y/N + date). Assign each to the fieldsets of the
categories listed in `docs/architecture.md`.

**Departments / users:** add real staff as users (or import them) before
checking assets out to people — never check out to free-text names; named
users are what make the history useful.

## 6. Barcode settings

1. Settings → Labels: choose barcode type (Code 128 is the standard pick for
   asset tags; QR for tags you want phone-scannable).
2. Asset tag format: Prefix + counter (e.g. `IT-1001`) — set the counter start
   per prefix and **never reuse** a retired tag.
3. Print one test label on your label printer, scan it into the asset lookup,
   confirm it resolves to the right record before printing a batch.
4. Label stock: direct-thermal for office gear; durable synthetic labels for
   warehouse devices that get handled and bumped.

## 7. Import the sample CSV

1. Rename `templates/assets-import.csv.example` → `.csv`.
2. In Snipe-IT: Assets → Import → upload, select "Asset" import type.
3. Map columns: Item Name → Asset Name, Category → Category, Model → Asset
   Model, Manufacturer → Manufacturer, Serial Number → Serial, Asset Tag →
   Asset Tag, Status → Status, Location → Location, Purchase Date / Purchase
   Cost → matching fields, Notes → Notes.
4. Run the import, then spot-check two rows against the CSV.

The sample rows are fictional (`SN-FAKE-####`). For a real migration, expect
to clean the source data first: dedupe rows, normalize category/model names
to the exact names created above, and resolve duplicate serials — the importer
flags them, but only you can decide which record is truth.

## 8. Optional: mail + notifications

If you filled in mail settings, send a test email from Settings → Mail and
enable: checkout/checkin notifications, overdue-asset reminders, and audit
notifications. Without mail, assign someone to watch the overdue-assets
report on a schedule instead.
